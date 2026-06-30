# RunService: Stepped (PreSimulation)

## 1. Core Architecture and Engine Definition

`RunService.Stepped` is a fundamental event in the Luau environment that operates on both the Client and the Server. It fires every single frame, specifically **immediately before the Roblox physics engine processes its simulations**. 

In the overarching architecture of the Roblox Engine Pipeline (which utilizes the Havok physics engine), the engine must resolve collisions, calculate rigid body dynamics, evaluate constraints (like hinges, springs, and ropes), and apply forces. `Stepped` acts as the critical injection point for developers to mutate physical properties right before the C++ backend calculates the physical state of the world for that specific frame.

Because it runs immediately prior to physics interpolation and simulation, it is the mathematically correct environment to apply custom physics logic, modify velocities, or intervene in standard gravity operations.

## 2. Technical Specifications

* **Execution Environment:** Dual-Scope (Available on both Server via `Script` and Client via `LocalScript`).
* **Yielding Behavior:** Synchronous / Blocking. The physics engine will halt and wait for all `Stepped` threads to finish executing before it proceeds with the collision and dynamics algorithms.
* **Passed Arguments:** Returns two highly specific arguments:
    1.  `time` (Type: `number`): The total elapsed time in seconds since the game instance began running (effectively the lifetime of the server or client session).
    2.  `deltaTime` (Type: `number`): The exact elapsed time in seconds since the *previous* `Stepped` event fired.
* *Critical Note on Arguments:* Unlike `RenderStepped` and `Heartbeat` which only pass `deltaTime`, `Stepped` historically passes the accumulated `time` as its first argument. Forgetting this distinction is a common source of bugs for developers transitioning between different RunService events.

## 3. The Task Scheduler and Physics Pipeline

To fully comprehend `Stepped`, you must visualize the precise moment it executes within the Task Scheduler's frame cycle:

1.  **Input Processing:** Client inputs are registered.
2.  **Network Processing:** Inbound remote events and property replications are processed.
3.  **`Stepped` / `PreSimulation` (Execution phase):** Custom Lua code fires here. 
4.  **Physics Simulation (Havok Engine):** * Broad-phase and Narrow-phase collision detection.
    * Constraint solving (AlignPosition, HingeConstraint, etc.).
    * Velocity and CFrame resolution based on applied forces.
5.  **Post-Simulation / Heartbeat:** The frame loop wraps up after physics are finalized.
6.  **Rendering:** (Client only) Visuals are drawn to the screen.

When you modify the `AssemblyLinearVelocity` of a part inside `Stepped`, the physics engine immediately sees that new velocity during Step 4 and calculates the trajectory appropriately. If you were to apply this change in `Heartbeat`, the physics engine would have already completed its math for the frame, meaning your velocity change would sit idle until the *next* frame's physics step, creating micro-stutters and desynchronization.

## 4. Mathematical Integration and Physics Engine Interaction

Roblox uses a semi-implicit Euler integration method to calculate object motion. During the physics step (Step 4), the engine approximates the next state of rigid bodies. The fundamental kinematic equation governing positional updates is:

$$p_{n+1} = p_n + v_n \Delta t + \frac{1}{2} a \Delta t^2$$

Where:
* $p_{n+1}$ is the new position.
* $p_n$ is the current position.
* $v_n$ is the current velocity (which you can dictate in `Stepped`).
* $a$ is the acceleration (resulting from forces like gravity or `VectorForce`).
* $\Delta t$ is the `deltaTime`.

By executing code in `Stepped`, you are explicitly defining $v_n$ and influencing $a$ exactly before the engine computes $p_{n+1}$. For advanced aerodynamics, developers often calculate drag force inside `Stepped` using the drag equation:

$$F_d = \frac{1}{2} \rho v^2 C_d A$$

Where $F_d$ is the opposing drag force applied to a body, $\rho$ is the fluid density, $v$ is the relative velocity, $C_d$ is the drag coefficient, and $A$ is the reference area. Calculating and applying this force in `Stepped` ensures the physics engine factors the drag into its acceleration integration flawlessly.

## 5. Modernization: `Stepped` vs `PreSimulation`

In modern Roblox development, Roblox has introduced a nomenclature shift to make the Task Scheduler more readable. 
* **`RunService.Stepped`** is the legacy name.
* **`RunService.PreSimulation`** is the modern equivalent.

Functionally and technically, both events map to the exact same execution point in the C++ backend pipeline. However, `PreSimulation` only returns `deltaTime` (1 argument) by default in its modern implementation, whereas `Stepped` returns both `time` and `deltaTime` (2 arguments) to maintain backwards compatibility with older scripts. Moving forward, Roblox engineering recommends conceptualizing this phase as the "Pre-Simulation" step.

## 6. Principal Use Cases (Best Practices)

1.  **Custom Physics Controllers:** Creating custom character movement systems (like Quake-style movement, wall-running, or surfing) where you directly manipulate `HumanoidRootPart.AssemblyLinearVelocity` bypassing default Humanoid physics.
2.  **Custom Gravity / Anti-Gravity Mechanisms:** Negating the workspace gravity for specific parts by calculating an exact upward force based on the part's mass ($F = m \cdot g$) and applying it right before the engine tries to pull the part downward.
3.  **Conveyor Belts and Moving Platforms:** Manually overriding the velocity of anchored or unanchored moving parts so that when the physics engine calculates collisions with a player standing on them, the friction and momentum carry the player correctly.
4.  **Network Ownership Verification:** Checking or forcing network ownership of unanchored parts immediately before they are simulated.

## 7. Anti-Patterns (Strictly Prohibited Uses)

To ensure server stability and client framerate optimization, NEVER perform the following inside `Stepped`:

1.  **Visual Updates or Camera Manipulation:** Modifying the Camera or UI in `Stepped` is an anti-pattern. Because physics has not yet simulated for the frame, pinning a UI or a Camera to a physical object here will result in a 1-frame visual lag. The camera will look at where the part *was*, not where it *will be* after physics finishes.
2.  **Post-Physics Raycasting:** If you want to raycast to see if a projectile hit a wall after moving, doing it in `Stepped` is wrong. You will be raycasting against the physics state of the *previous* frame. Raycasting for hit registration should generally be done in `Heartbeat` (PostSimulation).
3.  **Heavy Iteration on the Server:** Running a loop over thousands of parts in a server-sided `Stepped` event will stall the entire physics engine. If the `Stepped` thread takes longer than ~16ms on the server, server physics will slow down, resulting in "slow-motion" gameplay for all clients.
4.  **Yielding Mechanisms:** Never use `task.wait()`, `WaitForChild()`, or unresolved `Promises`. Yielding inside `Stepped` breaks the synchronous loop.

## 8. Advanced Implementation Examples

### Example 1: Custom Terminal Velocity / Air Drag (Server or Client)
This script applies a dampening force before physics simulates, preventing a part from falling faster than a specified terminal velocity.

```lua
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local fallingPart = Workspace:WaitForChild("FallingBoulder")
local MAX_FALL_SPEED = -50 -- Studs per second

-- Notice the two arguments for Stepped: time, deltaTime
local function applyAirFriction(totalTime: number, deltaTime: number)
    -- Verify the part exists and is physically active
    if not fallingPart or not fallingPart.Parent then return end

    local currentVelocity = fallingPart.AssemblyLinearVelocity
    
    -- If the object is falling faster than our limit on the Y axis
    if currentVelocity.Y < MAX_FALL_SPEED then
        -- We override the Y velocity immediately before physics simulation
        -- This prevents the engine from accelerating it further downward
        fallingPart.AssemblyLinearVelocity = Vector3.new(
            currentVelocity.X, 
            MAX_FALL_SPEED, 
            currentVelocity.Z
        )
    end
end

-- Connect to the event
local physicsConnection = RunService.Stepped:Connect(applyAirFriction)
Example 2: Perfect Custom Gravity (Hovering Object)
Using Stepped to constantly adapt a force to precisely counter gravity before the physics engine calculates the downward pull.

Lua
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local hoverPart = Workspace:WaitForChild("HoverDrone")

-- We use a VectorForce component to apply physics
local vectorForce = hoverPart:FindFirstChild("VectorForce")
if not vectorForce then
    vectorForce = Instance.new("VectorForce")
    vectorForce.Attachment0 = hoverPart:FindFirstChild("CenterAttachment") or Instance.new("Attachment", hoverPart)
    vectorForce.RelativeTo = Enum.ActuatorRelativeTo.World
    vectorForce.Parent = hoverPart
end

RunService.Stepped:Connect(function(totalTime: number, deltaTime: number)
    -- Mathematical formula for Weight: W = mass * gravity
    local mass = hoverPart.AssemblyMass
    local workspaceGravity = Workspace.Gravity
    
    -- Calculate the exact upward force needed to achieve neutral buoyancy
    local counterForce = mass * workspaceGravity
    
    -- Apply a slight sine wave to make it bob up and down over time
    local bobbingEffect = math.sin(totalTime * 2) * (mass * 5)
    
    -- Update the force right before Havok physics processes the body
    vectorForce.Force = Vector3.new(0, counterForce + bobbingEffect, 0)
end)
Example 3: Adapting to Modern PreSimulation
If you choose to use the modern naming convention, remember that it drops the time argument, returning only deltaTime.

Lua
local RunService = game:GetService("RunService")

-- PreSimulation is the modern alias/replacement for Stepped
RunService.PreSimulation:Connect(function(deltaTime: number)
    -- Ideal for physics steps where total elapsed time is irrelevant, 
    -- but delta integration is required.
    -- E.g., custom character controller movement integration.
end)