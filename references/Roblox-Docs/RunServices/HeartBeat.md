# RunService: Heartbeat (PostSimulation)

## 1. Core Architecture and Engine Definition

`RunService.Heartbeat` is a highly versatile, dual-scope event within the Luau environment that fires at the very end of every frame cycle. Crucially, it executes **after** the Roblox Havok physics engine has completed all of its collision detection, rigid body simulations, and constraint resolutions for that specific frame.

Because it operates at the tail-end of the pipeline, `Heartbeat` represents the "final state" of the world before the server replicates data to clients, or before the client prepares to render the frame. It is the most robust and commonly utilized event for general-purpose loops, state management, and continuous logic that does not explicitly need to precede physics (`Stepped`) or block rendering (`RenderStepped`).

## 2. Technical Specifications

* **Execution Environment:** Dual-Scope (Available on both Server via `Script` and Client via `LocalScript`).
* **Yielding Behavior:** Synchronous / Blocking. The engine will complete the execution of all `Heartbeat` connections before finalizing the frame cycle.
* **Passed Arguments:** Returns one argument:
    * `deltaTime` (Type: `number`): The exact elapsed time in seconds since the *previous* `Heartbeat` event fired.
* **Thread Synchronization:** Using `RunService.Heartbeat:Wait()` is the modern, highly optimized replacement for the deprecated `wait()` or `task.wait()` when you explicitly need to yield a thread until the current frame's physics simulations are completely resolved.

## 3. The Task Scheduler and Post-Physics Pipeline

Understanding `Heartbeat` requires identifying its exact placement within the Roblox Engine Task Scheduler:

1.  **Input & Network Processing:** Events, user inputs, and replication are ingested.
2.  **`Stepped` / `PreSimulation`:** Pre-physics logic executes.
3.  **Physics Simulation (Havok Engine):** The C++ backend calculates all positional changes, forces, and collisions. 
    * *Note: Objects have now officially moved to their new positions for the frame.*
4.  **`Heartbeat` / `PostSimulation` (Execution phase):** Custom Lua code fires here. 
    * *Because physics is complete, reading a part's `Position` or `CFrame` here guarantees you are reading its final, updated state for this frame.*
5.  **`RenderStepped` / `PreRender`:** (Client only) Visual/Camera updates execute.
6.  **Rendering:** (Client only) The final frame is drawn to the screen.

## 4. Mathematical Integration and State Interpolation

Because `Heartbeat` provides `deltaTime`, it is the standard environment for linear interpolation (Lerp) and evaluating kinematics that do not rely on the physics engine. For example, if you are scripting a custom, non-physical projectile (anchored), you must calculate its spatial translation manually. 

The positional vector formula for uniform motion is:

$$P_{final} = P_{initial} + (\vec{V} \cdot \Delta t)$$

Where:
* $P_{final}$ is the new `Vector3` position.
* $P_{initial}$ is the current `Vector3` position.
* $\vec{V}$ is the directional velocity vector.
* $\Delta t$ is the `deltaTime` passed by `Heartbeat`.

By executing this calculation in `Heartbeat`, you ensure that the projectile's movement is frame-rate independent. Whether the server is running at a degraded 20 FPS or a flawless 60 FPS, the mathematical distance traveled over real-world time remains absolutely consistent.

## 5. Modernization: `Heartbeat` vs `PostSimulation`

To improve the semantic clarity of the Task Scheduler, Roblox introduced a modern naming convention:
* **`RunService.Heartbeat`** is the legacy name, though still universally supported and heavily utilized.
* **`RunService.PostSimulation`** is the modern equivalent.

Functionally, technically, and under the hood, both events map to the exact same C++ execution phase. They both return `deltaTime`. Utilizing `PostSimulation` is entirely a matter of preference for developers who prefer the pipeline's naming convention (`PreRender`, `PreSimulation`, `PostSimulation`) to be uniform.

## 6. Principal Use Cases (Best Practices)

1.  **Hit Registration and Raycasting:** This is the most critical use case. Because `Heartbeat` fires *after* physics, raycasting to detect bullet hits here ensures you are evaluating against the exact, final positions of enemy hitboxes for that frame. Raycasting in `Stepped` would test against their positions from the *previous* frame.
2.  **General Game Loops & State Machines:** Checking if a match should end, evaluating if a player has captured a flag, or iterating through a continuous state machine (e.g., custom NPC AI logic).
3.  **Cooldowns and Timers:** Accumulating `deltaTime` to manage ability cooldowns or status effects (like poison damage over time) accurately.
4.  **Custom UI Tracking Physics:** If a 2D ScreenGui element needs to follow a 3D unanchored physical part (using `WorldToScreenPoint`), updating it in `Heartbeat` ensures the UI element perfectly matches the part's final physical position for that frame.
5.  **Non-Physical Animations:** Moving Anchored parts manually via CFrame manipulation.

## 7. Anti-Patterns (Strictly Prohibited Uses)

To maintain a healthy, bug-free codebase, avoid the following inside `Heartbeat`:

1.  **Applying Physics Forces:** Do not modify `AssemblyLinearVelocity`, `VectorForce`, or apply impulses here. Because the physics simulation phase has already passed (Step 3), the physics engine will not process your changes until the *next* frame. This causes a 1-frame latency/desync in movement. Always use `Stepped` for pre-physics mutations.
2.  **Camera Manipulation:** Do not update the player's Camera CFrame here. If you update the camera in `Heartbeat`, but the player moves their mouse between `Heartbeat` and `RenderStepped`, the camera will feel "floaty" or laggy. Camera math belongs exclusively in `RenderStepped`.
3.  **Heavy Synchronous Blocking:** Running deeply nested loops over thousands of instances in `Heartbeat` will drastically increase the frame time, forcing the server or client to drop frames. If computing complex AI pathfinding, distribute the workload across multiple frames or use Parallel Luau (`task.desynchronize()`).
4.  **Infinite Yielding:** Never place `task.wait()` or unresolved asynchronous calls inside a `Heartbeat` connection.

## 8. Advanced Implementation Examples

### Example 1: High-Fidelity Custom Projectile (Raycasting)
This example demonstrates why `Heartbeat` is strictly required for hit registration. It calculates continuous movement and raycasts *after* all physical characters have updated their positions.

```lua
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

-- Configuration
local PROJECTILE_SPEED = 500 -- Studs per second
local projectileCFrame = CFrame.new(0, 5, 0)
local projectileDirection = Vector3.new(0, 0, -1) -- Moving forward

-- Raycast parameters to ignore the shooter
local raycastParams = RaycastParams.new()
raycastParams.FilterType = Enum.RaycastFilterType.Exclude
raycastParams.FilterDescendantsInstances = {} 

local function simulateProjectile(deltaTime: number)
    -- 1. Calculate how far the projectile travels this exact frame
    local distanceThisFrame = PROJECTILE_SPEED * deltaTime
    local movementVector = projectileDirection * distanceThisFrame
    
    -- 2. Raycast from current position to the projected next position
    -- Doing this in Heartbeat ensures we hit targets that JUST moved via physics
    local result = Workspace:Raycast(projectileCFrame.Position, movementVector, raycastParams)
    
    if result then
        -- Hit detected!
        print("Projectile hit:", result.Instance.Name, "at", result.Position)
        
        -- Apply damage logic here...
        
        -- Disconnect/Destroy projectile
        -- (Assuming this function is connected to a variable we can disconnect)
        return true 
    end
    
    -- 3. If no hit, advance the projectile's position for the next frame
    projectileCFrame = projectileCFrame + movementVector
    
    -- (Optional) Update visual tracer Part CFrame here
    return false
end

local connection
connection = RunService.Heartbeat:Connect(function(deltaTime: number)
    local hasHit = simulateProjectile(deltaTime)
    if hasHit then
        connection:Disconnect()
    end
end)
Example 2: Accurate Timers and Cooldown Accumulation
Instead of using while task.wait(1) do, professional developers accumulate deltaTime in Heartbeat for precise state management.

Lua
local RunService = game:GetService("RunService")

local COOLDOWN_DURATION = 5.0 -- 5 seconds
local timeAccumulator = 0
local isAbilityReady = false

RunService.Heartbeat:Connect(function(deltaTime: number)
    if not isAbilityReady then
        -- Accumulate the elapsed time
        timeAccumulator += deltaTime
        
        -- Check if the threshold is met
        if timeAccumulator >= COOLDOWN_DURATION then
            isAbilityReady = true
            timeAccumulator = 0
            print("Ability is now ready to use!")
            
            -- Trigger UI updates or visual effects here
        end
    end
end)
Example 3: Modern PostSimulation Syntax
Utilizing the modern nomenclature for continuous non-physical rotation.

Lua
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local radarDish = Workspace:WaitForChild("RadarDish") -- Anchored part
local ROTATION_SPEED_RAD = math.rad(90) -- 90 degrees per second

-- PostSimulation is mechanically identical to Heartbeat
RunService.PostSimulation:Connect(function(deltaTime: number)
    -- Rotate the anchored part flawlessly regardless of frame rate
    local rotationThisFrame = ROTATION_SPEED_RAD * deltaTime
    radarDish.CFrame = radarDish.CFrame * CFrame.Angles(0, rotationThisFrame, 0)
end)