# RunService
> Roblox Luau reference for frame phases, simulation timing, and loop control.
> Use this file as the single RunService guide for AI reasoning.

## 1. Purpose

RunService controls execution timing across the Roblox frame cycle.
It is the base for logic that depends on simulation order, physics timing, or render timing.

Use it when you want:
- per-frame gameplay control
- physics-adjacent logic
- camera or visual updates
- frame-rate independent motion
- environment checks like server, client, or studio

Do not use it as a replacement for `task.wait()` or for heavy blocking work.

## 2. Core Principles

1. Execution order matters.
2. `RenderStepped` is client-only.
3. `PreSimulation` is for pre-physics changes.
4. `PostSimulation` is for post-physics reads.
5. `Heartbeat` is the legacy name for post-simulation flow.
6. Use `deltaTime` for frame-rate independence.
7. Every connection must be cleaned up.
8. Prefer event-based logic over polling.
9. Keep frame loops thin.
10. Use the correct phase for the job, not the most familiar one.

## 3. Frame Phases

| Phase | Modern Name | Legacy Name | Best Use |
|---|---|---|---|
| before physics | `PreSimulation` | `Stepped` | forces, velocity changes, physics setup |
| after physics | `PostSimulation` | `Heartbeat` | reading final world state, timers, raycasts |
| before render | `PreRender` | `RenderStepped` | camera, UI, visual polish |

## 4. API Surface

### 4.1 PreSimulation

Use for:
- velocity changes
- force changes
- physics preparation
- server-authoritative movement correction

Avoid:
- camera updates
- post-hit raycasts
- long loops
- yielding

### 4.2 PostSimulation

Use for:
- reading final positions
- hit registration
- timers and cooldowns
- anchored motion
- state evaluation after physics

Avoid:
- physics force mutation
- camera updates
- heavy synchronous work

### 4.3 PreRender / RenderStepped

Use for:
- camera control
- UI tracking
- visual interpolation
- cosmetic effects

Avoid:
- server logic
- physics mutation
- heavy calculations

## 5. Environment Checks

- `RunService:IsClient()` for client-only logic
- `RunService:IsServer()` for server-only logic
- `RunService:IsStudio()` for editor-specific behavior
- `RunService:IsRunning()` for runtime checks

## 6. Correct Mapping

### Physics control

- modify motion in `PreSimulation`
- read final motion in `PostSimulation`

### Camera and UI

- update in `PreRender` or `RenderStepped`

### Server gameplay

- validate on server-side simulation phases

### Visual polish

- keep client-only rendering logic separate from gameplay logic

## 7. DeltaTime Rule

`deltaTime` is the time passed since the last frame phase.
Use it for:
- movement
- rotation
- timers
- cooldowns
- interpolation

Never assume fixed frame rate.

## 8. Lifecycle Rule

Any frame connection should have a teardown path.

Rules:
- store the connection
- disconnect on destroy or stop
- avoid global unmanaged connections
- avoid connection explosion

## 9. Manager Pattern

For larger games, one frame connection should fan out to registered systems.

Rules:
- keep one central update entry point
- register systems with the manager
- distribute work intentionally
- avoid one `Heartbeat` connection per object

This keeps frame logic scalable and easier to clean up.

## 10. Performance Rules

- do not block frame loops
- do not run expensive scans every frame
- do not use `RenderStepped` for gameplay
- prefer signals over polling
- spread heavy work across frames
- throttle optional visuals when full frame rate is not needed

## 11. Security Rules

- never trust client-side movement math
- validate important movement and state on the server
- treat client rendering as cosmetic only
- keep authoritative checks outside render-only flow

## 12. Common Failures

### 12.1 Wrong phase

Problem: physics mutation happens after physics.
Fix: move it to `PreSimulation`.

### 12.2 Camera in the wrong place

Problem: camera feels delayed or floaty.
Fix: move it to `PreRender` or `RenderStepped`.

### 12.3 Blocking the loop

Problem: frame time spikes from heavy logic.
Fix: split work or move it out of the loop.

### 12.4 Leaked connections

Problem: updates continue after the system ends.
Fix: disconnect on teardown.

## 13. Professional Behavior Rules for AI

- pick the phase that matches the job
- prefer simple timing logic
- keep frame code short
- warn when a loop is in the wrong phase
- separate gameplay timing from visual timing

## 14. Minimal Checklist

- correct phase chosen
- `deltaTime` used for time-based motion
- connections cleaned up
- no heavy work in render phase
- no physics mutation after physics
- server authority preserved

## 15. Final Rule

RunService is about timing discipline.
Use the right phase, keep the loop thin, and let the engine breathe.

