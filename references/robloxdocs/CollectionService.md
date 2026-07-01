# CollectionService
> Roblox Luau reference for tag-driven architecture, lifecycle control, replication, and clean component orchestration.
> Use this file as the single CollectionService guide for AI reasoning.

## 1. Purpose

CollectionService is a Roblox engine service for tagging Instances with string labels.
It is the base for data-driven systems that need stable instance grouping without relying on folders, names, or class checks.

Use it when you want:
- instance discovery by role
- decoupled component bootstrapping
- automatic cleanup when an instance is removed or destroyed
- server-authoritative tagging with client-side read access

Do not use it as a message bus, a datastore, or a replacement for security validation.

## 2. Core Principles

1. Tags are identifiers, not game state.
2. Server tags are authoritative.
3. Client tags are local-only and must never be trusted for security decisions.
4. `GetTagged()` is a snapshot, not a live list.
5. `GetInstanceAddedSignal()` and `GetInstanceRemovedSignal()` are the correct lifecycle hooks.
6. Every attach path must have a matching detach path.
7. Build registries for scale, not repeated scanning.
8. Treat tag-driven systems as components with explicit ownership.

## 3. API Surface

### 3.1 AddTag

`CollectionService:AddTag(instance, tag)`

- Adds a tag to an instance.
- Safe to call more than once, but duplicate intent should still be prevented by your code.
- Server-side tags replicate.
- Client-side tags stay local.

### 3.2 RemoveTag

`CollectionService:RemoveTag(instance, tag)`

- Removes a tag if it exists.
- Fires removed lifecycle signals.

### 3.3 HasTag

`CollectionService:HasTag(instance, tag)`

- Fast boolean check.
- Safe for targeted runtime checks.
- Use this for validation, not for broad discovery loops.

### 3.4 GetTags

`CollectionService:GetTags(instance)`

- Returns a snapshot of all tags on one instance.
- Use for inspection or debugging.

### 3.5 GetTagged

`CollectionService:GetTagged(tag)`

- Returns a snapshot list of all instances with that tag.
- Avoid repeated use in hot loops.

### 3.6 GetInstanceAddedSignal

`CollectionService:GetInstanceAddedSignal(tag)`

- Fires when an instance gains the tag.
- Pair with `GetTagged()` for full bootstrap coverage.

### 3.7 GetInstanceRemovedSignal

`CollectionService:GetInstanceRemovedSignal(tag)`

- Fires when the tag is removed or the instance is destroyed.
- Use it to tear down state, connections, UI, and cached references.

## 4. Replication Rules

| Action | Server | Client |
|---|---|---|
| AddTag | authoritative, replicates | local-only |
| RemoveTag | authoritative, replicates | local-only |
| HasTag | reads server state | reads local state |
| GetTagged | server snapshot | local snapshot |

Rules:
- Only the server should create gameplay-critical tag state.
- Client tags are acceptable for UI, VFX, prediction, and local-only decoration.
- Never let client-reported tag state decide damage, rewards, access, or anti-cheat logic.
- Streaming can delay client visibility, so client snapshots are not guaranteed to match server state at a moment in time.

## 5. Correct Lifecycle Pattern

The correct flow is:
1. bootstrap existing tagged instances with `GetTagged()`
2. connect `GetInstanceAddedSignal()`
3. connect `GetInstanceRemovedSignal()`
4. detach cleanly on removal or destroy

That order prevents missed instances and prevents ghost state.

### Standard pattern

```lua
local CollectionService = game:GetService("CollectionService")

local TAG = "Interactable"

local function onAdded(instance: Instance)
    -- setup
end

local function onRemoved(instance: Instance)
    -- cleanup
end

for _, instance in CollectionService:GetTagged(TAG) do
    task.spawn(onAdded, instance)
end

CollectionService:GetInstanceAddedSignal(TAG):Connect(onAdded)
CollectionService:GetInstanceRemovedSignal(TAG):Connect(onRemoved)
```

Use `task.spawn` for bootstrap when the handler can yield or may fail independently.

## 6. Registry Pattern

The strongest pattern for large systems is a live registry keyed by `Instance`.
This avoids repeated `GetTagged()` calls and gives O(1) lookup for the active set.

Rules:
- keep a table of live instances
- update it in added and removed signals
- treat it as the authoritative runtime cache
- never rebuild the same list every frame

The registry pattern is the right answer when a system must query many tagged instances repeatedly.

## 7. Component Pattern

CollectionService becomes powerful when each tag maps to a component module.
Each component should own one role and expose a small contract:

- `attach(instance)`
- `detach(instance)`

The manager owns registration, bootstrapping, and signal wiring.
The component owns the logic for one tagged instance.

### Contract rules

- attach and detach must be idempotent-safe
- attach should not depend on hidden global state
- detach must tolerate destroyed instances
- per-instance state must be stored by instance key, not in shared module variables
- yield-heavy work should be deferred

### Component shape

```lua
local Component = {}
local active = {}

function Component.attach(instance: Instance)
    if active[instance] then return end
    active[instance] = true
end

function Component.detach(instance: Instance)
    if not active[instance] then return end
    active[instance] = nil
end

return Component
```

Keep the shape simple and predictable.

## 8. Manager Pattern

A CollectionManager is a custom orchestration layer on top of CollectionService.
It is not built into Roblox.

Use it when:
- one tag maps to one component
- many tags need centralized bootstrapping
- you want one loader to own registration
- you need error isolation across components

Manager responsibilities:
- register each tag once
- attach existing tagged instances at boot
- connect future added and removed signals
- keep a live active map
- warn on duplicate registration
- isolate component errors with protected calls

Manager non-responsibilities:
- game design
- security policy
- event routing between systems
- tag naming decisions

The manager should stay boring and reliable.

## 9. Event Routing Pattern

CollectionService handles instance lifecycle.
It does not replace event systems.

Use an event bus when systems need to communicate about gameplay actions instead of instance presence.

Correct division:
- CollectionService: what exists and how it is tagged
- EventSystem: what happened
- RemoteEvent / RemoteFunction: cross-boundary communication

This separation keeps code professional and prevents direct coupling between systems.

## 10. Tag Query Rules

CollectionService does not provide built-in complex filters.
If you need AND or OR logic, implement it explicitly.

Rules:
- `HasTag` for one-instance checks
- `GetTagged` for initial set discovery
- custom helper functions for multi-tag logic
- maintain your own filtered registry for repeated queries

Do not assume the engine will do complex composition for you.

## 11. Tag Naming

Use namespaced, readable tags.

Good:
- `Combat_Enemy_Active`
- `World_Prop_Destructible`
- `UI_Button_Highlighted`

Bad:
- `enemy`
- `tag1`
- `temp`

Rules:
- be explicit
- avoid collisions
- keep tag meaning stable across systems
- centralize tag strings in a registry module

Never hardcode tag strings all over the codebase.

## 12. Tag Registry

Always define tag names in one constants table.

Principles:
- no inline string littering
- no guessing tag spelling in random scripts
- all systems should read the same source of truth
- renaming a tag should require one edit point

This is one of the simplest ways to keep a project large and maintainable.

## 13. Cleanup Rules

Any connection created during attach must be disconnected during detach.
Any per-instance table entry must be cleared.
Any long-running task tied to that instance must be canceled or made harmless.

Important:
- `GetInstanceRemovedSignal()` can fire because the tag was removed
- `GetInstanceRemovedSignal()` can also fire because the instance was destroyed
- detach must not assume the instance is fully intact

If a component uses nested connections, the removed handler must clean all of them.

## 14. Timing and Streaming

Streaming changes when clients see instances, but not the core rule.

Rules:
- server remains authoritative
- client signals can arrive later than server state changes
- client snapshots can be incomplete
- do not trust client startup snapshots for complete world state

Design systems so they tolerate delayed visibility.

## 15. Performance Rules

Use these as hard defaults:
- prefer signals over polling
- prefer registries over repeated scans
- avoid `GetTagged()` in per-frame loops
- avoid storing mutable shared state in module upvalues
- avoid duplicate signal connections
- use `pcall` around risky component boundaries

The best CollectionService architecture is the one that scales without needing constant rescans.

## 16. Security Rules

Security is simple here:
- server decides real tag state
- client may request an action, not assert ownership
- client tags may support visuals only
- server should re-check instance validity before important operations

Never use tags from the client as proof of permission, identity, or damage eligibility.

## 17. Common Failures

### 17.1 Missing bootstrap

Problem: existing tagged instances never receive setup.
Fix: always run `GetTagged()` first.

### 17.2 Duplicate attach

Problem: the same instance gets multiple handlers.
Fix: guard by instance state in the component or manager.

### 17.3 Missing detach

Problem: memory leaks, stacked connections, ghost logic.
Fix: make removal handling mandatory.

### 17.4 Hot-loop scanning

Problem: repeated `GetTagged()` causes unnecessary cost.
Fix: keep a live registry.

### 17.5 Client trust mistake

Problem: client tag state affects gameplay authority.
Fix: validate on server.

### 17.6 Hardcoded strings

Problem: tag spelling drifts across files.
Fix: central tag registry.

## 18. Professional Behavior Rules for AI

When generating CollectionService-based systems, the AI should:
- choose the simplest correct architecture first
- separate setup, runtime, and cleanup
- prefer readable ownership over cleverness
- keep logic small and traceable
- avoid unnecessary abstraction
- treat tags as contracts, not decoration
- respect server authority
- preserve performance by default

The goal is not to look advanced.
The goal is to be correct, stable, and easy to extend.

## 19. Recommended Mental Model

Think in three layers:
- Tag layer: marks what an instance is
- Registry layer: tracks what is currently active
- Component layer: performs the actual behavior

If a design violates this layering, it usually becomes harder to maintain.

## 20. Minimal Checklist

Before shipping any CollectionService feature:
- tag names are centralized
- existing instances are bootstrapped
- future instances are handled
- removed instances clean up fully
- server owns important state
- client-only tags are clearly non-authoritative
- no repeated `GetTagged()` in hot paths
- no duplicate handlers
- no leaked connections

## 21. Final Rule

CollectionService should make the project more organized, not more mysterious.
Use it to make instance-driven systems clean, observable, and maintainable.

