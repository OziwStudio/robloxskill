# RunService: RenderStepped

## 1. Core Architecture and Engine Definition

`RunService.RenderStepped` is a client-exclusive event in the Luau environment that fires every single frame, exactly before the Roblox engine pushes the frame to the GPU for rendering. 

In the context of the Roblox Engine Pipeline, the engine operates on a continuous loop of updating inputs, processing physics, updating internal states, and finally drawing the visual representation on the screen. `RenderStepped` acts as the final programmable gateway before the "Draw" call. 

Because it operates at this critical juncture, it holds absolute synchronous authority over the camera and client-sided visual mutations. The engine *must* wait for all active `RenderStepped` threads to yield or complete before it can advance the rendering pipeline. This makes it both extremely powerful for frame-perfect visual synchronization and extremely dangerous for performance degradation.

## 2. Technical Specifications

*   **Execution Environment:** Client-Side Only (`LocalScript` or a `ModuleScript` required by a `LocalScript`). Attempting to connect to this event on the Server will result in an error.
*   **Yielding Behavior:** Strictly Synchronous / Blocking. The engine rendering thread halts until the execution of the bound functions completes.
*   **Passed Arguments:** `deltaTime` (Type: `number`). 
    *   This represents the exact elapsed time in seconds since the *previous* `RenderStepped` event fired.
    *   It is critical to use `deltaTime` for any math inside this event to ensure frame-rate independence (e.g., an animation plays at the same speed whether the client is running at 30 FPS, 60 FPS, or 144 FPS).

## 3. The Task Scheduler and Rendering Pipeline

To understand `RenderStepped`, one must understand where it sits in the Roblox Task Scheduler. A standard frame on the client follows this highly simplified execution order:

1.  **Input Processing:** Mouse, Keyboard, Gamepad, and Touch inputs are registered.
2.  **Network Processing:** Inbound replication from the Server is processed.
3.  **Pre-Simulation / Stepped:** Early frame logic.
4.  **Physics Simulation:** The engine calculates rigid body dynamics, collisions, and constraints.
5.  **Post-Simulation / Heartbeat:** Post-physics logic, state updates, and general loop processing.
6.  **Humanoid State Updates:** Animations and joint transformations are applied.
7.  **`RenderStepped` (Execution phase):** This is where Lua code attached to `RenderStepped` fires.
8.  **Render / Draw Call:** The GPU processes the Draw Job and outputs pixels to the monitor.

Because `RenderStepped` fires *after* physics and animations, but *before* the draw call, it is the only place to correct positional data (like locking a custom weapon viewmodel to the camera) without visual tearing, lag, or "jitter" that would occur if evaluated earlier in the pipeline.

## 4. Performance Economics and The Frame Budget

Because `RenderStepped` is blocking, it introduces strict performance budgets:
*   At **60 FPS**, you have approximately **16.67 milliseconds** (ms) to compute everything in the *entire frame* (Physics, Network, Scripts, Render).
*   At **144 FPS** (using an unlocked framerate), you have only **6.94 ms**.

If the code inside `RenderStepped` takes 10 ms to execute, the engine will inevitably drop frames because the GPU is starved waiting for the Lua thread to finish. 
*   **Profiling Rule of Thumb:** Code in `RenderStepped` should ideally execute in `< 0.5 ms`. You can measure this using the Roblox MicroProfiler (Ctrl+F6). Look for the `Render` and `PreRender` labels.

## 5. `:Connect()` vs `:BindToRenderStep()`

For advanced development, using `RunService.RenderStepped:Connect(function)` is often considered bad practice when managing multiple complex visual systems. Instead, the engine provides `RunService:BindToRenderStep()`.

### Why use BindToRenderStep?
`:Connect()` attaches functions arbitrarily. You cannot guarantee the order in which multiple connected functions will execute. If your Camera script and your Viewmodel script both use `:Connect()`, the Viewmodel might update its position *before* the Camera updates its rotation, resulting in a 1-frame delay (jitter).

`BindToRenderStep(name: string, priority: number, function)` solves this by introducing a strict execution queue based on integer priorities.

### The RenderPriority Enumeration
Roblox provides the `Enum.RenderPriority` standard to establish a logical hierarchy:
1.  **`Enum.RenderPriority.First.Value` (100):** Executes at the absolute beginning of the render step.
2.  **`Enum.RenderPriority.Input.Value` (200):** Executes when input is processed. VR tracking and custom cursor logic should run here.
3.  **`Enum.RenderPriority.Camera.Value` (300):** This is the exact priority where the default Roblox Camera updates. Custom camera controllers MUST bind at or slightly above this priority (e.g., 299 or 301 depending on the goal).
4.  **`Enum.RenderPriority.Character.Value` (400):** Character IK (Inverse Kinematics) and default animations update here.
5.  **`Enum.RenderPriority.Last.Value` (2000):** Executes immediately before the screen draws. Viewmodels (First-Person arms) should bind here (e.g., `Camera.Value + 1` or `Last.Value`) to guarantee they react to the final camera CFrame.

## 6. Principal Use Cases (Best Practices)

1.  **Custom Camera Controllers:** (e.g., OTS - Over The Shoulder cameras, RTS fixed-perspective cameras, or First-Person forced cameras).
2.  **First-Person Viewmodels:** Updating the CFrame of an arm/weapon model to perfectly match `workspace.CurrentCamera.CFrame` multiplied by an offset.
3.  **Dynamic Screen-Space UI:** Pinning a 3D object to the screen using `Camera:ScreenPointToRay()` or maintaining a custom 3D crosshair.
4.  **Procedural Animations reliant on Camera:** E.g., Viewmodel sway, weapon bobbing, or screen shake math.

## 7. Anti-Patterns (Strictly Prohibited Uses)

To maintain a healthy codebase and high FPS, NEVER do the following inside `RenderStepped`:

1.  **Physics Manipulation:** Do not apply `VectorForce`, `BodyVelocity`, or update `.Velocity` properties here. The engine's physics step has already passed. Physics mutations here will be ignored until the *next* frame, causing desynchronization. Use `PreSimulation` or `Heartbeat`.
2.  **State Management & Cooldowns:** Do not use this event to check if a player's ability cooldown has expired, or to regenerate health. Use `Heartbeat`.
3.  **Heavy Iteration (Looping through workspace):** Iterating through thousands of parts or checking complex distances here will destroy framerates.
4.  **Network Calls:** Never fire a `RemoteEvent` or `RemoteFunction` bound to `RenderStepped`. You will instantly throttle the client's network bandwidth and likely get disconnected by the server.
5.  **Yielding Mechanisms:** Never use `task.wait()`, `WaitForChild()`, or unresolved `Promises` inside the connected function.

## 8. Advanced Implementation Examples

### Example 1: Proper Binding with Priorities for a Viewmodel
```lua
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local Camera = Workspace.CurrentCamera
local Viewmodel = Workspace:WaitForChild("Viewmodel") -- Assuming a predefined rig

-- A unique identifier for the binding to prevent memory leaks and duplication
local BIND_NAME = "WeaponViewmodelUpdate"

local function UpdateViewmodel(deltaTime: number)
    -- Clamp deltaTime to prevent massive jumps after a severe lag spike
    local dt = math.clamp(deltaTime, 0, 0.1)

    -- Example sway calculation based on time
    local sway = math.sin(time() * 3) * 0.02
    local swayOffset = CFrame.new(sway, sway, 0)
    
    -- The Viewmodel updates AFTER the camera has finalized its math for the frame
    Viewmodel:PivotTo(Camera.CFrame * swayOffset * CFrame.new(0.5, -0.5, -1))
end

-- Binding with a priority higher than the Camera (Execute AFTER Camera updates)
local priority = Enum.RenderPriority.Camera.Value + 10 
RunService:BindToRenderStep(BIND_NAME, priority, UpdateViewmodel)

-- To disconnect later (e.g., when un-equipping the weapon):
-- RunService:UnbindFromRenderStep(BIND_NAME)
Example 2: Frame-Rate Independent Lerping (Springs/Damping)
When interpolating values visually inside RenderStepped, you must account for deltaTime so that the transition speed remains identical across different framerates.

Lua
local RunService = game:GetService("RunService")

local currentRecoil = 0
local targetRecoil = 5
local LERP_SPEED = 10 -- Speed multiplier

RunService.RenderStepped:Connect(function(deltaTime: number)
    -- Using an exponential decay function for smooth, frame-independent interpolation
    -- Formula: a + (b - a) * (1 - math.exp(-decay * dt))
    
    local blend = 1 - math.exp(-LERP_SPEED * deltaTime)
    currentRecoil = currentRecoil + (targetRecoil - currentRecoil) * blend
    
    -- Apply currentRecoil to camera or weapon...
end)