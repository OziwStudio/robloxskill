# TweenService
> Roblox Luau reference for interpolation, motion design, and smooth transitions.
> Use this file as the single TweenService guide for AI reasoning.

## 1. Purpose

TweenService interpolates Instance properties over time.
It is the standard tool for smooth motion, UI transitions, and polished visual feedback.

Use it when you want:
- state changes that feel smooth
- UI entrance and exit motion
- object movement or rotation
- cosmetic feedback with timing

Do not use it for:
- physics simulation
- game-state authority
- frame-by-frame procedural logic

## 2. Core Principles

1. Tweens are visual helpers, not gameplay authority.
2. Tween state should be explicit and manageable.
3. Overlapping tweens on the same property can conflict.
4. `TweenInfo` is configuration, not state.
5. Cleanup matters for every tween you create.
6. Client-side tweens are best for responsive visuals.
7. Use the simplest motion that solves the problem.

## 3. API Surface

### 3.1 Create

`TweenService:Create(instance, tweenInfo, propertyTable)`

- creates a tween object
- does not play automatically
- validates target properties before use

### 3.2 Play

`tween:Play()`

- starts or resumes playback

### 3.3 Pause

`tween:Pause()`

- pauses at the current state

### 3.4 Cancel

`tween:Cancel()`

- stops the tween
- use when state changes or the owner is destroyed

### 3.5 Completed

`tween.Completed`

- fires when the tween finishes or is cancelled
- useful for cleanup and sequencing

## 4. TweenInfo

`TweenInfo` defines how the motion behaves.

| Parameter | Purpose |
|---|---|
| Time | duration |
| EasingStyle | curve shape |
| EasingDirection | curve direction |
| RepeatCount | repeat amount |
| Reverses | reverse after finish |
| DelayTime | wait before start |

Rules:
- define `TweenInfo` once when possible
- use the easing curve that matches the motion
- do not recreate tween settings in a tight loop

## 5. Replication Rules

| Action | Server | Client |
|---|---|---|
| property change | replicates if the property is replicated | local only for client-owned visuals |
| tween logic | can run on either side | can run on either side |
| visual smoothness | may be limited by replication | usually best on client |

Rules:
- use the server when the tween affects gameplay state
- use the client when the tween is only cosmetic
- do not trust tween result as authority

## 6. Lifecycle Pattern

The correct flow is:
1. define the tween config
2. create the tween
3. connect `Completed` if cleanup is needed
4. play it
5. cancel or destroy it when the owner changes

### Standard pattern

```lua
local TweenService = game:GetService("TweenService")

local info = TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
local tween = TweenService:Create(script.Parent, info, {Transparency = 1})

tween.Completed:Connect(function()
    tween:Destroy()
end)

tween:Play()
```

## 7. Active Tween Rule

For properties that can be retargeted often, keep one active tween per object.

Rules:
- cancel the old tween before starting a new one
- destroy finished tweens
- track active tweens in a registry when needed
- prevent tween stacking on the same property

## 8. Chaining Rule

Keep chaining simple.

Rules:
- use one step at a time
- avoid deeply nested callbacks
- wait for completion only when the next step depends on it

## 9. Performance Rules

- avoid tweening huge batches of objects at once
- avoid unnecessary tween churn
- do not tween unsupported or meaningless properties
- destroy tween objects after use
- reduce off-screen or low-value motion when possible

## 10. Security Rules

- do not use tween state as proof of gameplay state
- do not use client visual motion as server authority
- the server should verify actual state, not animation intent

## 11. Common Failures

### 11.1 Tween stacking

Problem: several tweens fight over the same property.
Fix: cancel or replace the old tween first.

### 11.2 Missing cleanup

Problem: tween objects stay around after completion.
Fix: destroy them after `Completed`.

### 11.3 Destroyed owner

Problem: tween target no longer exists.
Fix: verify the instance before creating the tween.

### 11.4 Invalid property

Problem: property cannot be tweened.
Fix: only use tweenable properties.

## 12. Advanced Notes

- use client-side tweens for UI and VFX responsiveness
- use server-side tweens when the motion affects gameplay state
- use camera tweens only when the camera is set up correctly
- use other interpolation methods when tweening cannot express the motion cleanly

## 13. Professional Behavior Rules for AI

- prefer readable motion over clever chaining
- always manage cleanup
- choose the simplest TweenInfo first
- warn about stacking conflicts
- use client-side execution for pure UI/VFX when appropriate

## 14. Minimal Checklist

- `TweenInfo` is defined clearly
- active tweens are managed
- `Completed` handles cleanup
- tween target exists
- property is tweenable
- side of execution is correct
- no stacking conflict

## 15. Final Rule

TweenService should make motion feel polished and controlled.
If tween management becomes too large, move it into a dedicated animation layer.

