# RunService: PreSimulation

## 1. Core Architecture and Engine Definition

`RunService.PreSimulation` is the modern, formalized event within the Luau environment that dictates the exact moment **immediately before the Roblox physics engine evaluates the world state**. Operating on both the Server and the Client, it is the modern replacement and exact pipeline equivalent of the legacy `Stepped` event, updated to align with current engine nomenclature and cleaner argument passing.

In the context of the Roblox Engine, physical simulations (handled by the proprietary integration of the Havok physics engine) are not continuous; they are discrete mathematical steps calculated every frame. The engine must evaluate the current velocities, masses, and applied forces of all unanchored parts, solve for collisions, and then move them to their new positions. `PreSimulation` is the critical, programmable gateway where a developer can alter these physical properties *right before* the engine performs those calculations.

By injecting logic into `PreSimulation`, developers ensure that their custom physics modifications are seamlessly integrated into the engine's native physics step, preventing micro-stutters, desynchronization, or physics anomalies.

## 2. Technical Specifications

* **Execution Environment:** Dual-Scope. Fully supported on both the Server (`Script`) and the Client (`LocalScript`).
* **Yielding Behavior:** Strictly Synchronous / Blocking. The C++ physics engine completely halts and waits for all Lua threads connected to `PreSimulation` to finish executing before it proceeds to calculate collisions and rigid body dynamics.
* **Passed Arguments:** Returns exactly one argument:
    * `deltaTime` (Type: `number`): The elapsed time in seconds since the *previous* `PreSimulation` event fired.
* **Modernization Note vs. Stepped:** The legacy `Stepped` event returns two arguments (`time` and `deltaTime`). `PreSimulation` deliberately drops the `time` argument to streamline the API and match the single-argument signature of `PostSimulation` and `PreRender`. 

## 3. The Modern Task Scheduler and Physics Pipeline

To master `PreSimulation`, one must understand its exact placement within the modern Roblox Task Scheduler. A single frame execution follows this chronological order:

1.  **Input Registration:** Mouse, keyboard, and gamepad states are polled.
2.  **Network Ingestion:** Remote events and replication data from the server/client are received.
3.  **`PreSimulation` (Execution Phase):** Custom Lua code fires here. This is the last chance to modify the physical state (velocity, forces) before the engine takes over.
4.  **Havok Physics Simulation (The Engine Step):**
    * *Broad-phase Collision:* Quickly culls objects that are too far apart to collide.
    * *Narrow-phase Collision:* Calculates exact geometric intersection points.
    * *Constraint Solver:* Evaluates springs, hinges, ropes, and custom constraints.
    * *Integration:* Updates the `CFrame` of all unanchored parts based on their velocities.
5.  **`PostSimulation`:** Logic that evaluates the *results* of the physics step executes here.
6.  **Humanoid Kinematics:** Animations and joint transformations apply.
7.  **`PreRender` & Draw Call:** (Client only) Visuals are finalized and pushed to the GPU.

## 4. Mathematical Integration and the Havok Engine

Roblox utilizes Semi-Implicit Euler integration to calculate kinematics. When the physics engine runs (Step 4), it calculates the new velocity and position of an object based on the forces applied to it. The fundamental equations are:

$$v_{n+1} = v_n + (a \cdot \Delta t)$$
$$p_{n+1} = p_n + (v_{n+1} \cdot \Delta t)$$

Where:
* $v_{n+1}$ is the new velocity.
* $v_n$ is the current velocity (which you can modify in `PreSimulation`).
* $a$ is acceleration (derived from $F = m \cdot a$).
* $p_{n+1}$ is the new position.
* $\Delta t$ is the `deltaTime`.

If you want to apply a custom aerodynamic drag force, the formula is:

$$F_{drag} = -\frac{1}{2} \cdot \rho \cdot v^2 \cdot C_d \cdot A$$

By calculating $F_{drag}$ and applying it to a part's `AssemblyLinearVelocity` or a `VectorForce` inside `PreSimulation`, the Havok engine incorporates your exact math into its $v_{n+1}$ calculation seamlessly. If you did this in `PostSimulation`, the engine would have already moved the part, causing a one-frame delay in the drag application.

## 5. Network Ownership and Determinism

`PreSimulation` plays a vital role in game security and determinism. Roblox uses a distributed physics model called "Network Ownership." If a client owns a part, the client simulates its physics and tells the server where it is. If the server owns it, the server dictates the physics.

If you are scripting a server-authoritative anti-cheat or a custom server-sided projectile system, running the velocity validation logic inside the server's `PreSimulation` ensures that the server's physics engine overrides any fraudulent client replication *before* the server mathematically processes the frame and replicates it to other clients.

## 6. Principal Use Cases (Best Practices)

1.  **Custom Character Controllers:** Bypassing the default `Humanoid` to create physics-based movement, such as wall-running, sliding, or bunny-hopping. You dictate the exact `AssemblyLinearVelocity` of the `HumanoidRootPart` here.
2.  **Aerodynamics and Fluid Dynamics:** Applying custom air resistance, terminal velocity clamping, or buoyancy forces to parts before they fall or sink.
3.  **Custom Gravity Systems:** If creating a game with planetary gravity (spherical gravity fields), you calculate the gravitational pull vector towards the planet's core and apply it in `PreSimulation`.
4.  **Vehicle Suspension Systems:** Raycasting downward from a vehicle's chassis to calculate suspension spring compression (Hooke's Law: $F = -k \cdot x$), then applying that corrective upward force right before the physics step.
5.  **Conveyor Belts / Moving Platforms:** Manually injecting velocity into anchored parts so that when unanchored parts (like players) rest on them, the physics engine calculates the friction and moves the player accordingly.

## 7. Anti-Patterns (Strictly Prohibited Uses)

To maintain server health, high FPS, and prevent visual desynchronization, **NEVER** do the following inside `PreSimulation`:

1.  **Visual or Camera Updates:** Modifying a player's Camera `CFrame` or updating UI elements here will result in visual tearing. Because the physics step hasn't happened yet, the camera will look at where an object *was*, not where it *is going to be* this frame. Always use `PreRender` for the camera.
2.  **Hit Registration (Raycasting for Damage):** If you raycast a bullet path in `PreSimulation`, you are testing against the hitboxes from the *previous* frame. Physics hasn't updated them yet. Always use `PostSimulation` for hit detection.
3.  **Heavy Server Iteration:** Looping through thousands of `Workspace` descendants here will drastically block the physics engine. If `PreSimulation` takes >10ms on the Server, the entire server's physics will slow down, resulting in a "slow-motion" effect for all players.
4.  **Yielding:** Using `task.wait()`, `WaitForChild()`, or asynchronous network calls will break the synchronous loop and cause catastrophic script behavior.

## 8. Advanced Implementation Examples

### Example 1: Aerodynamic Drag / Terminal Velocity Control
This script applies a dampening force based on the current velocity, ensuring an object doesn't accelerate infinitely downwards, seamlessly integrated before Havok calculates the drop.

```lua
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local fallingPart = Workspace:WaitForChild("DropPod")
local DRAG_COEFFICIENT = 0.05

RunService.PreSimulation:Connect(function(deltaTime: number)
    if not fallingPart or not fallingPart.Parent then return end

    local currentVelocity = fallingPart.AssemblyLinearVelocity
    
    -- Calculate speed squared for realistic drag scaling
    local speed = currentVelocity.Magnitude
    if speed > 0.1 then
        -- Drag force opposes the direction of movement
        local dragDirection = -currentVelocity.Unit
        local dragForceMagnitude = (speed ^ 2) * DRAG_COEFFICIENT
        
        -- We modify velocity directly before the engine calculates gravity integration
        local deceleration = (dragDirection * dragForceMagnitude * deltaTime) / fallingPart.AssemblyMass
        fallingPart.AssemblyLinearVelocity = currentVelocity + deceleration
    end
end)
Example 2: Planetary (Spherical) Custom Gravity
This applies a gravitational pull towards a specific point in space, overriding the default downward -Y gravity.

Lua
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local PlanetCore = Workspace:WaitForChild("PlanetCore")
local Player = game.Players.LocalPlayer
local GRAVITY_STRENGTH = 50 -- Adjust based on desired pull

RunService.PreSimulation:Connect(function(deltaTime: number)
    local character = Player.Character
    if not character then return end
    
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end
    
    -- 1. Get the directional vector from the player to the planet's core
    local directionToCore = (PlanetCore.Position - rootPart.Position).Unit
    
    -- 2. Calculate the force required (F = m * a)
    local mass = rootPart.AssemblyMass
    local gravityForce = directionToCore * GRAVITY_STRENGTH * mass
    
    -- 3. We must ensure Workspace.Gravity = 0 or we apply a counter-force to default gravity
    -- Assuming default gravity is disabled for this part:
    
    -- We can use a VectorForce component updated every frame
    local vectorForce = rootPart:FindFirstChild("PlanetGravityForce")
    if vectorForce then
        vectorForce.Force = gravityForce
    end
end)
Example 3: Safe Velocity Capping (Anti-Fling Mechanism)
Roblox physics can sometimes experience math explosions (flinging) when two constraints fight. You can use PreSimulation to forcefully clamp velocities before the engine acts on them.

Lua
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local MAX_SAFE_VELOCITY = 300 -- Studs per second
local myVehicle = Workspace:WaitForChild("PhysicsCar")

RunService.PreSimulation:Connect(function(deltaTime: number)
    -- Iterate through the physical assemblies of the vehicle
    for _, part in ipairs(myVehicle:GetDescendants()) do
        if part:IsA("BasePart") and not part.Anchored then
            local vel = part.AssemblyLinearVelocity
            
            -- If the physics engine was about to process a massive velocity spike, we cap it
            if vel.Magnitude > MAX_SAFE_VELOCITY then
                part.AssemblyLinearVelocity = vel.Unit * MAX_SAFE_VELOCITY
                print("Prevented a physics fling on:", part.Name)
            end
            
            local rotVel = part.AssemblyAngularVelocity
            if rotVel.Magnitude > 50 then
                part.AssemblyAngularVelocity = rotVel.Unit * 50
            end
        end
    end
end)