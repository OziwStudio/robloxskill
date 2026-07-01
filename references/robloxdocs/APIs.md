# Roblox Engine APIs
> Standard reference for Roblox engine API usage, safety, performance, and maintainability.
> Use this file as the primary API reasoning guide for AI output.

## 1. Purpose

This document defines how to use Roblox engine APIs in a professional way.
The goal is not only code that works, but code that is stable, safe, and scalable.

Use this file when the task involves:
- engine services
- remotes
- persistence
- input
- scheduling
- loop behavior
- object lifecycle
- defensive coding

## 2. Core Principles

1. Use `game:GetService()` for services.
2. Avoid fragile direct indexing for services.
3. Treat every client input as untrusted.
4. Treat every network or persistence call as fallible.
5. Prefer `task` library over legacy `wait` and `spawn`.
6. Avoid expensive scans inside per-frame loops.
7. Clean up every connection you create.
8. Set properties before parenting instances when possible.
9. Keep API use explicit and readable.
10. Prefer server authority for game-critical decisions.

## 3. Service Access

### Rule

Always resolve services with `game:GetService("ServiceName")`.

### Reason

- it is explicit
- it is standard
- it avoids brittle service lookup
- it keeps code readable across environments

### Good

```lua
local TweenService = game:GetService("TweenService")
local DataStoreService = game:GetService("DataStoreService")
```

### Avoid

```lua
local TweenService = game.TweenService
```

## 4. DataStoreService

DataStore is a persistence boundary, not a casual table store.
It should be treated as critical infrastructure.

### Rules

- cache data in memory first
- write only when needed
- use `UpdateAsync` for transactional changes
- use `SetAsync` only when transactional safety is not needed
- wrap calls in `pcall`
- retry with backoff when appropriate
- respect throttling

### Pattern

```lua
local DataStoreService = game:GetService("DataStoreService")
local store = DataStoreService:GetDataStore("PlayerStats")

local function savePlayerData(userId: number, data)
    local ok, err = pcall(function()
        store:UpdateAsync(tostring(userId), function()
            return data
        end)
    end)

    if not ok then
        warn("Save failed:", err)
    end
end
```

### Notes

- use one cached table per player or entity
- flush on shutdown, leave, or safe checkpoints
- never spam writes from Heartbeat

## 5. Networking

Networking is the client-server boundary.
It must be validated, rate-limited, and narrow.

### Rules

- validate every remote input on the server
- rate-limit remote calls
- use `RemoteEvent` for one-way actions
- use `RemoteFunction` only when a return value is truly needed
- verify player, character, distance, state, and permission
- never trust client damage, currency, or ownership claims

### Security Checklist

- player verified
- instance verified
- distance verified
- state verified
- permission verified
- cooldown verified
- payload type verified

### Principle

Client requests intent.
Server decides outcome.

## 6. Task Library

The `task` library is the modern scheduling standard.

### Rules

- use `task.spawn()` for immediate asynchronous work
- use `task.defer()` for bootstrapping and end-of-cycle scheduling
- use `task.wait()` instead of `wait()`
- use `task.delay()` for timed future work

### Timing Rule

- use `task.defer()` when initialization order matters
- use `task.spawn()` when work should start now
- use `task.wait()` for modern delay behavior

## 7. Input APIs

Use input APIs based on the level of abstraction needed.

### ContextActionService

Use for gameplay actions that should work across keyboard, touch, and controller.

### UserInputService

Use for raw input detection, device checks, and lower-level interaction.

### Rule

- prefer abstract actions for gameplay
- prefer raw input only when abstraction is not enough
- branch by platform only when behavior truly differs

## 8. RunService

RunService controls update timing.

### Heartbeat

- post-physics
- good for timers, general updates, and sync logic

### Stepped

- pre-physics
- good for physics-adjacent updates

### RenderStepped

- client only
- pre-render
- good for camera and visual polish

### Rule

- never run expensive logic in RenderStepped
- never use frame loops for work that signals can solve

## 9. Instance Lifecycle

Objects should be created and cleaned up deliberately.

### Rules

- create the instance first
- set properties before parenting
- store connections for later cleanup
- call `:Destroy()` for cleanup
- avoid `:Remove()`
- avoid leaving orphaned references

### Pattern

```lua
local part = Instance.new("Part")
part.Size = Vector3.new(4, 1, 4)
part.Anchored = true
part.Parent = workspace
```

## 10. Memory Rules

Memory discipline keeps systems stable over long sessions.

### Rules

- avoid global mutable state
- nil out unused references
- clear tables when they are no longer needed
- avoid holding Instances longer than necessary
- use `table.create()` when the size is known
- disconnect every event connection

## 11. Defensive Coding

Errors are part of runtime reality.

### Rules

- use `pcall` for risky engine calls
- use `xpcall` when you need safer error handling or better tracing
- never assume external APIs always succeed
- always plan a fallback or recovery path

### Pattern

```lua
local ok, result = pcall(function()
    return someFunction()
end)

if not ok then
    warn("API call failed:", result)
end
```

## 12. Modernization Checklist

Use this when auditing existing code:

- all services use `GetService`
- `wait()` replaced by `task.wait()`
- `spawn()` replaced by `task.spawn()` or `task.defer()`
- risky calls wrapped in `pcall`
- instance parents set last
- server validates remote actions
- connections cleaned up on destroy
- Luau types used where they improve clarity

## 13. Professional AI Behavior

When generating code with these APIs:

- choose the simplest correct API
- add comments only where they help reasoning
- warn when usage is risky or non-standard
- prefer explicit server authority
- keep performance in mind by default
- avoid relying on hidden engine behavior

## 14. Common Failures

### 14.1 Parent First

Problem: parenting before setup can trigger premature signals or flicker.

Fix: set properties first, parent last.

### 14.2 Connection Leak

Problem: connections remain active after the owner is gone.

Fix: store connections and disconnect on teardown.

### 14.3 DataStore Spam

Problem: repeated reads or writes cause throttling and instability.

Fix: cache locally and save in controlled intervals.

### 14.4 Remote Trust

Problem: client sends value claims directly to server.

Fix: server calculates outcomes from trusted state.

### 14.5 Per-Frame Scanning

Problem: expensive scans in Heartbeat or RenderStepped.

Fix: use signals, caches, or registries.

## 15. Final Rule

APIs are tools.
Architecture decides whether those tools stay safe, fast, and maintainable.
