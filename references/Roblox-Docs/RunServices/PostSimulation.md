# RunService: PostSimulation

## 1. Core Architecture and Engine Definition

`RunService.PostSimulation` is the modern, formalized event within the Luau environment that fires at the exact conclusion of the physics step. It operates on both the Server and the Client and represents the **final world state** of the frame before the server replicates data to clients or the client begins its rendering pipeline.

This event is the direct, modern equivalent of the legacy `Heartbeat` event. In the Roblox Engine's execution pipeline, the proprietary Havok physics engine takes a discrete step every frame to calculate collisions, resolve constraints, and update the positions of all moving rigid bodies. `PostSimulation` is the programmable execution phase that occurs immediately *after* the C++ engine has finished these calculations.

Because objects have officially moved to their new positions for the frame when this event fires, it is the absolute source of truth for reading spatial data, raycasting, and evaluating the results of physical interactions.

## 2. Technical Specifications

* **Execution Environment:** Dual-Scope. Available and fully supported on both the Server (`Script`) and the Client (`LocalScript`).
* **Yielding Behavior:** Synchronous / Blocking. The engine will halt and complete the execution of all `PostSimulation` connections before finalizing the frame cycle.
* **Passed Arguments:** Returns exactly one argument:
    * `deltaTime` (Type: `number`): The exact elapsed time in seconds since the *previous* `PostSimulation` event fired.
* **Thread Synchronization:** Using `RunService.PostSimulation:Wait()` is the highly optimized, engine-native method to yield a thread until the current frame's physics simulations are completely resolved.

## 3. The Modern Task Scheduler and Post-Physics Pipeline

To architect robust systems, developers must conceptualize `PostSimulation` within the modern chronological frame cycle:

1.  **Input Registration:** Client inputs (mouse, keyboard) are polled.
2.  **Network Ingestion:** Inbound replication is processed.
3.  **`PreSimulation`:** Developers inject custom physics forces or velocity overrides here.
4.  **Havok Physics Simulation:** * The engine calculates $F = m \cdot a$, resolves collisions, and integrates velocity into positional changes. 
    * *Critically: Unanchored parts have now officially moved to their new coordinates.*
5.  **`PostSimulation` (Execution Phase):** Custom Lua code fires here. Developers read the new positions and evaluate game state.
6.  **Humanoid Kinematics & Animation:** Motor6Ds update.
7.  **`PreRender` & Draw Call:** (Client only) Visuals update and pixels are drawn to the screen.

## 4. Mathematical Integration and Kinematics

Because `PostSimulation` provides `deltaTime`, it is the engine's standard environment for manual kinematic calculations (like linear interpolation or moving anchored parts) that do not rely on the Havok physics engine. 

For example, when scripting uniform motion for a custom, anchored projectile, you calculate its spatial translation using the velocity vector and delta time:

$$P_{n+1} = P_n + (\vec{v} \cdot \Delta t)$$

Where:
* $P_{n+1}$ is the next positional `Vector3`.
* $P_n$ is the current positional `Vector3`.
* $\vec{v}$ is the constant directional velocity vector.
* $\Delta t$ is the `deltaTime`.

Evaluating this in `PostSimulation` ensures that non-physical motion is completely frame-rate independent and synchronized chronologically with the rest of the game's physics evaluation.

## 5. Modernization: `PostSimulation` vs `Heartbeat`

* **`RunService.Heartbeat`** is the legacy naming convention.
* **`RunService.PostSimulation`** is the modern nomenclature.

Under the hood of the C++ task scheduler, both events are bound to the exact same execution signal. They are functionally identical. However, Roblox engineering introduced `PostSimulation` to unify the API semantics. Structuring an entire game framework using `PreRender`, `PreSimulation`, and `PostSimulation` provides a mathematically logical and highly readable lifecycle architecture compared to the arbitrary legacy terms.

## 6. Principal Use Cases (Best Practices)

1.  **Hit Registration and Raycasting:** This is the most vital use case. Raycasting for bullet impacts or melee hitboxes MUST be done here. Because physics just finished, testing a ray against the workspace in `PostSimulation` guarantees you are evaluating against the exact, final positions of enemy hitboxes for the current frame.
2.  **State Machines and Game Loops:** General game logic (e.g., "Is the flag captured?", "Has the match timer run out?", "Is the NPC in range of the player?") should execute here, as it evaluates the finalized world state.
3.  **Cooldown and Buff Accumulators:** Using the `deltaTime` argument to accurately reduce cooldown timers or apply damage-over-time (DOT) effects.
4.  **Tracking Physical Objects with UI:** Updating a 2D ScreenGui element's position to follow an unanchored 3D part. Executing in `PostSimulation` ensures the UI perfectly tracks the part's final position for that frame before the screen draws.
5.  **Anchored Part Animation:** Rotating or moving anchored objects via CFrame manipulation (e.g., spinning coins, opening doors, moving platforms).

## 7. Anti-Patterns (Strictly Prohibited Uses)

To prevent severe latency, desynchronization, and performance drops, **NEVER** do the following inside `PostSimulation`:

1.  **Applying Physics Forces:** Do not modify `AssemblyLinearVelocity`, `VectorForce`, or apply impulses here. Because the Havok physics step has already passed (Step 4), any forces applied here will sit idle and won't be calculated until the *next* frame. This introduces a 1-frame mechanical latency. Always use `PreSimulation` for forces.
2.  **Camera Manipulation:** Do not update the `Workspace.CurrentCamera` CFrame here. If you modify the camera in `PostSimulation`, and the player moves their mouse *after* this step but *before* `PreRender`, the camera will feel floaty and stuttery.
3.  **Heavy Synchronous Iteration:** Looping through complex AI pathfinding for hundreds of NPCs in a single `PostSimulation` frame will stall the server. Use Parallel Luau (`task.desynchronize()`) or distribute the workload across multiple frames.

## 8. Advanced Implementation Examples

### Example 1: Authoritative Projectile Raycasting
This script demonstrates the necessity of `PostSimulation` for accurate hit detection. By raycasting *after* physics, we hit targets exactly where they are currently rendered.

```lua
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local PROJECTILE_SPEED = 800 -- Studs per second
local currentPos = Vector3.new(0, 10, 0)
local direction = Vector3.new(0, 0, -1) -- Forward

local raycastParams = RaycastParams.new()
raycastParams.FilterType = Enum.RaycastFilterType.Exclude
raycastParams.FilterDescendantsInstances = {} 

local connection
connection = RunService.PostSimulation:Connect(function(deltaTime: number)
    -- Calculate movement vector for this specific frame
    local stepDistance = PROJECTILE_SPEED * deltaTime
    local movementVector = direction * stepDistance
    
    -- Raycast against the post-physics world state
    local result = Workspace:Raycast(currentPos, movementVector, raycastParams)
    
    if result then
        print("Hit registered on:", result.Instance.Name)
        -- Handle damage/effects here
        
        -- Terminate the simulation
        connection:Disconnect()
    else
        -- Advance the theoretical position
        currentPos = currentPos + movementVector
        
        -- (Optional: Update a cosmetic tracer part's CFrame to currentPos here)
    end
end)
Example 2: Frame-Rate Independent Anchored Rotation
Rotating an object seamlessly without relying on the physics engine.

Lua
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local floatingCrystal = Workspace:WaitForChild("Crystal") -- Anchored part
local ROTATION_SPEED_RAD = math.rad(45) -- 45 degrees per second

RunService.PostSimulation:Connect(function(deltaTime: number)
    -- The rotation distance is scaled perfectly to the elapsed time
    local rotationThisFrame = ROTATION_SPEED_RAD * deltaTime
    
    -- Multiply the CFrame to apply relative rotation
    floatingCrystal.CFrame = floatingCrystal.CFrame * CFrame.Angles(0, rotationThisFrame, 0)
end)
Example 3: Yielding for State Verification
Instead of using task.wait(), you can yield exactly until the engine finalizes the physics step to ensure safe data reading.

Lua
local RunService = game:GetService("RunService")

local function performPostPhysicsCheck()
    -- We want to read the position of a part, but we want to be 
    -- 100% sure the physics engine isn't currently moving it.
    
    -- Yield the thread until PostSimulation fires
    local deltaTime = RunService.PostSimulation:Wait()
    
    -- Physics is guaranteed to be finalized for this frame
    -- It is now mathematically safe to read spatial data
    print("Physics step complete, delta time was:", deltaTime)
end

task.spawn(performPostPhysicsCheck)