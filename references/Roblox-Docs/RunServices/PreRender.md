Markdown
# RunService: PreRender

## 1. Core Architecture and Engine Definition

`RunService.PreRender` is the modern, formalized event in the Luau environment that represents the absolute final execution step before the client renders a frame. Functioning exclusively on the Client, it is the direct, updated nomenclature for the legacy `RenderStepped` event. 

In the Roblox Engine Pipeline, the client operates in a continuous loop: polling input, replicating network data, processing physics, calculating Humanoid kinematics, executing standard game logic, and finally, generating a 2D image from the 3D world to push to the GPU. `PreRender` is the programmable threshold immediately preceding that final GPU draw call.

Because of its position at the extreme end of the frame cycle, it has supreme authority over the final visual state of the client's screen. Any changes made to visual elements (Camera, ViewportFrames, UI) within `PreRender` are guaranteed to be pushed to the monitor in that exact same frame, ensuring zero-latency visual synchronization.

## 2. Technical Specifications

* **Execution Environment:** Client-Side Only (`LocalScript` or a client-sided `ModuleScript`). Server execution will instantly error, as the server operates headless (without a rendering pipeline or GPU draw loop).
* **Yielding Behavior:** Strictly Synchronous / Blocking. The engine's rendering thread completely halts. The frame cannot be drawn until every function connected to `PreRender` has successfully returned.
* **Passed Arguments:** Returns one argument:
    * `deltaTime` (Type: `number`): The elapsed time in seconds since the *previous* `PreRender` event fired.
* **Engine Parity:** Under the hood of the C++ Task Scheduler, `PreRender` is bound to the exact same execution signal as `RenderStepped`.

## 3. The Task Scheduler and the Modernization Shift

Roblox introduced `PreRender` alongside `PreSimulation` and `PostSimulation` to unify the Task Scheduler's naming conventions, moving away from the arbitrary "Stepped" terminology inherited from legacy physics terminology. 

The modern Client Frame Pipeline is structured as follows:
1.  **Input Registration:** Mouse/Keyboard state is captured.
2.  **`PreSimulation` (formerly Stepped):** Pre-physics logic executes.
3.  **Havok Physics Simulation:** Collision and constraint logic.
4.  **`PostSimulation` (formerly Heartbeat):** Post-physics game state logic.
5.  **Humanoid & Animation Step:** Bone transforms and motor6Ds update based on animation tracks.
6.  **`PreRender` (formerly RenderStepped):** Final visual overrides execute here.
7.  **Draw Call:** The projection matrix is sent to the GPU.

*Note on Prioritization:* While `PreRender` serves as the standard event signal, the engine still maintains `RunService:BindToRenderStep()` for complex systems (like default character controllers) that require explicit execution ordering via `Enum.RenderPriority`.

## 4. Mathematical Integration: CFrame and Projection Matrices

When you manipulate the `Workspace.CurrentCamera` inside `PreRender`, you are mathematically altering the transformation matrix that the engine uses to calculate the View Matrix. 

In Roblox, a `CFrame` is a $4 \times 4$ transformation matrix containing both positional and rotational data. The matrix is structured as:

$$M = \begin{bmatrix} r_{11} & r_{12} & r_{13} & p_x \\ r_{21} & r_{22} & r_{23} & p_y \\ r_{31} & r_{32} & r_{33} & p_z \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

Where:
* The $3 \times 3$ sub-matrix of $r$ components represents the rotational unit vectors (Right, Up, Look).
* The vector $\begin{bmatrix} p_x & p_y & p_z \end{bmatrix}^T$ represents the translation (Position in 3D space).

By calculating and applying modifications to this matrix inside `PreRender`, you ensure that the subsequent projection math (converting the 3D frustum into 2D screen-space pixels) utilizes your exact custom vectors before the GPU rasterizes the image.

## 5. MicroProfiler Analysis and Frame Budgets

Because `PreRender` is a blocking operation, it holds the rendering thread hostage. This introduces the most critical performance bottleneck in front-end Roblox development.
* To maintain **60 FPS**, the entire frame budget (Steps 1 through 7) is exactly **16.67ms**.
* To maintain **144 FPS**, the budget shrinks to **6.94ms**.

If you open the Roblox MicroProfiler (Ctrl+F6), the execution time of your connected functions will appear under the `PreRender` tag. 
* **Strict Rule:** The cumulative execution time of all `PreRender` code should never exceed **0.5ms to 1.0ms**. If your `PreRender` operations take 5ms, you are consuming nearly a third of the client's frame budget just on visual overrides, which guarantees framerate drops on lower-end hardware.

## 6. Principal Use Cases (Modern Best Practices)

1.  **Zero-Latency UI Tracking:** Utilizing `Camera:WorldToScreenPoint()` to pin a 2D `ScreenGui` (like an ESP box, custom name tag, or hitmarker) to a 3D physical object. Executing this in `PreRender` ensures the UI never lags behind the physical part's position on screen.
2.  **Custom Crosshairs and Reticles:** Adjusting dynamic crosshair spread based on player movement or recoil just before the frame draws.
3.  **Real-Time ViewportFrame Updates:** If you have a 3D mini-map or a character preview in a `ViewportFrame`, updating the internal `ViewportFrame` camera inside `PreRender` ensures it perfectly mirrors the main world camera.
4.  **Procedural Camera Shakers:** Applying mathematical noise (like Perlin noise or spring dampening) to the camera's CFrame to simulate explosions, walking head-bob, or weapon recoil.

## 7. Anti-Patterns (Strictly Prohibited Uses)

Violating these constraints in `PreRender` is the leading cause of memory leaks, client crashes, and severe FPS degradation:

1.  **Instantiating or Destroying Objects:** Never use `Instance.new()` or `Part:Destroy()` inside `PreRender`. Memory allocation and garbage collection are incredibly expensive operations that will spike the MicroProfiler and cause massive stuttering.
2.  **Raycasting (Unless Camera-Specific):** Avoid heavy hit-detection raycasting here. While a single raycast to check if the camera is clipping through a wall is acceptable, raycasting for bullet collisions must be done in `PostSimulation`.
3.  **Iterating Over the Workspace:** Using loops like `for _, obj in pairs(workspace:GetDescendants())` inside `PreRender` will freeze the client entirely.
4.  **Network Communication:** Firing a `RemoteEvent:FireServer()` in `PreRender` means you are spamming the server 60+ times a second. This will throttle the client's network queue and trigger server-side anti-spam disconnects.
5.  **State Logic & Timers:** Do not calculate game logic (e.g., "Is the player dead?", "Has the match started?") here. Use `PostSimulation`.

## 8. Advanced Implementation Examples

### Example 1: World-to-Screen UI Pinning (Zero-Latency)
This script demonstrates how to pin a 2D UI element perfectly to a 3D part without any visual delay, utilizing `PreRender`.

```lua
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local Players = game:GetService("Players")

local Camera = Workspace.CurrentCamera
local TargetPart = Workspace:WaitForChild("TargetObjective")

local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")
local TrackingUI = PlayerGui:WaitForChild("TrackingScreen"):WaitForChild("Marker")

local function updateMarkerPosition()
    -- Ensure the target still exists
    if not TargetPart then return end

    -- Project the 3D position into 2D screen space
    local screenPosition, onScreen = Camera:WorldToScreenPoint(TargetPart.Position)

    if onScreen then
        -- The object is in front of the camera; show and update the UI
        TrackingUI.Visible = true
        TrackingUI.Position = UDim2.fromOffset(screenPosition.X, screenPosition.Y)
        
        -- Optional: Scale UI based on depth (Z axis)
        -- local scale = 1000 / screenPosition.Z
        -- TrackingUI.Size = UDim2.fromOffset(scale, scale)
    else
        -- The object is behind the camera; hide the UI
        TrackingUI.Visible = false
    end
end

-- Connect to PreRender for flawless visual synchronization
RunService.PreRender:Connect(updateMarkerPosition)
Example 2: Safe DeltaTime Clamping for Procedural Animation
When using deltaTime for procedural animations (like spring-based weapon sway), lag spikes can cause massive deltaTime values, resulting in math explosions (NaN errors or the camera spinning wildly). This example shows how to clamp deltaTime safely inside PreRender.

Lua
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local Camera = Workspace.CurrentCamera

-- Variables for procedural bobbing
local walkTimeAccumulator = 0
local BOB_SPEED = 12
local BOB_INTENSITY = 0.5

-- We only want the bobbing to occur when the character is actually moving
local function getCharacterVelocity()
    local character = game.Players.LocalPlayer.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        local velocity = character.HumanoidRootPart.AssemblyLinearVelocity
        -- Ignore Y velocity (falling/jumping) for head bob
        return Vector3.new(velocity.X, 0, velocity.Z).Magnitude
    end
    return 0
end

RunService.PreRender:Connect(function(deltaTime: number)
    -- CRITICAL SAFEGURAD: Clamp deltaTime
    -- If the client freezes for 2 seconds, we treat it as max 0.1 seconds
    -- to prevent mathematical anomalies in our sine waves.
    local safeDeltaTime = math.clamp(deltaTime, 0, 0.1)
    
    local speed = getCharacterVelocity()
    
    if speed > 1 then
        -- Player is walking, accumulate time
        walkTimeAccumulator += safeDeltaTime
        
        -- Calculate vertical sine wave bob
        local bobY = math.sin(walkTimeAccumulator * BOB_SPEED) * BOB_INTENSITY
        -- Calculate horizontal cosine wave bob (half speed for figure-8 pattern)
        local bobX = math.cos(walkTimeAccumulator * (BOB_SPEED / 2)) * (BOB_INTENSITY / 2)
        
        -- Apply the transformation matrix to the camera
        local bobOffset = CFrame.new(bobX, bobY, 0)
        Camera.CFrame = Camera.CFrame * bobOffset
    else
        -- Smoothly reset accumulator when standing still to avoid snapping
        walkTimeAccumulator = 0
    end
end)