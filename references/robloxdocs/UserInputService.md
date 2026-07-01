# UserInputService
> Roblox Luau reference for client input, device detection, and cross-platform interaction.
> Use this file as the single UserInputService guide for AI reasoning.

## 1. Purpose

UserInputService handles raw player input on the client.
It is the foundation for custom controls, device-aware UI, and responsive interaction systems.

Use it when you want:
- keyboard, mouse, touch, or gamepad input
- device capability checks
- raw movement or camera control
- cross-platform input mapping

Do not use it for:
- server authority
- security decisions
- simple action binding when `ContextActionService` is enough

## 2. Core Principles

1. Input starts on the client, but important results must be server-validated.
2. `gameProcessedEvent` is the first filter for UI-safe input.
3. Input state can disappear if focus changes.
4. Events are for triggers; polling is for held state.
5. Every input connection must be cleaned up.
6. Cross-platform support is mandatory, not optional.

## 3. API Surface

### 3.1 InputBegan

Fires when input starts.

Use for:
- trigger actions
- key presses
- touch starts
- controller button presses

### 3.2 InputEnded

Fires when input ends.

Use for:
- hold release
- sprint end
- charge release
- drag end

### 3.3 InputChanged

Fires when active input changes.

Use for:
- mouse movement
- analog stick movement
- touch drag
- wheel input

Keep this path lightweight.

### 3.4 IsKeyDown

Checks whether a key is held.

Use for:
- continuous movement
- custom hold logic
- polling-based control states

### 3.5 GetMouseLocation

Returns current mouse position on screen.

Use for:
- custom cursors
- drag logic
- screen-space targeting

## 4. gameProcessedEvent Rule

Always check `gameProcessedEvent` first.

Rules:
- return immediately if it is true
- let UI and text input win unless override is intentional
- do not let gameplay steal typing or button focus

### Standard pattern

```lua
local UserInputService = game:GetService("UserInputService")

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end

    if input.KeyCode == Enum.KeyCode.M then
        toggleMenu()
    end
end)
```

## 5. Device Detection

Use runtime checks to adapt UI and control schemes.

Rules:
- detect touch, keyboard, mouse, and gamepad support
- react to controller connect and disconnect events
- do not assume one device type means another is missing
- keep prompts and actions device-aware

## 6. Lifecycle Pattern

Input connections must have a clear enable and disable path.

Rules:
- store every connection
- disconnect on teardown
- reset held states when focus changes
- avoid permanent unmanaged listeners

## 7. Input Object Rules

The input object gives context, not authority.

Useful fields:
- `UserInputType`
- `KeyCode`
- `UserInputState`
- `Position`
- `Delta`

Rules:
- read properties and discard the object
- do not store input objects permanently
- use `Delta` for high-frequency movement tracking

## 8. Touch Rules

Touch is not the same as mouse.

Rules:
- design for taps and drags separately
- use touch-friendly UI when possible
- support gestures only when needed
- do not assume every touch has a key code

## 9. ContextActionService vs UserInputService

Use `ContextActionService` when:
- you need automatic mobile buttons
- an action should map cleanly across devices
- action priority matters

Use `UserInputService` when:
- you need raw input
- you need device detection
- you need custom camera or movement control
- you need lower-level input state

## 10. Performance Rules

- exit early by input type
- avoid heavy math inside `InputChanged`
- avoid raycasts directly in noisy input events
- move continuous work to a render or simulation loop when needed
- clean up unused connections

## 11. Security Rules

- client input is a request, not proof
- never trust the client to claim a kill, hit, or reward
- server should validate important consequences
- rate-limit repeated input-driven requests

## 12. Common Failures

### 12.1 Ignoring `gameProcessedEvent`

Problem: typing in chat triggers gameplay.
Fix: return when processed is true.

### 12.2 Missing `InputEnded`

Problem: hold state gets stuck after focus loss.
Fix: reset state on focus change and cleanup.

### 12.3 PC-only controls

Problem: one key path is hardcoded.
Fix: map actions across keyboard, touch, and gamepad.

### 12.4 Heavy `InputChanged`

Problem: input lag and stutter.
Fix: keep the handler thin.

## 13. Professional Behavior Rules for AI

- validate `gameProcessedEvent` by default
- provide cross-platform mapping
- keep input state inside a manager or component
- use `RunService` for continuous motion
- manage enable and disable cleanly
- survive focus loss safely

## 14. Minimal Checklist

- UI typing does not trigger gameplay
- gamepad is mapped
- mobile has a valid path
- held input resets safely
- no heavy work in `InputChanged`
- connections are cleaned up

## 15. Final Rule

UserInputService connects player intent to game action.
Keep it responsive, cross-platform, and respectful of the UI layer above it.

