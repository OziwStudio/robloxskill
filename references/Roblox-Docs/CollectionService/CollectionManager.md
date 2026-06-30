# CollectionManager — CollectionService Reference
> Part of: `CollectionService` · Roblox Luau · Production Reference
> Scope: Centralized tag lifecycle management, component bootstrapping, registry orchestration, cleanup, error catalog

---

## 1. WHAT IS COLLECTIONMANAGER

`CollectionManager` is a **custom orchestration layer** built on top of `CollectionService`.
It does not exist as a Roblox built-in — it is an architectural pattern you implement.

**Purpose**: Instead of scattering `GetInstanceAddedSignal` / `GetInstanceRemovedSignal` calls
across dozens of scripts, `CollectionManager` provides a **single registry** that maps tag strings
to handler modules (components), bootstraps existing instances, and guarantees clean teardown.

**Why it matters**:
- Prevents duplicated signal connections across scripts.
- Enforces consistent attach/detach lifecycle for every tagged component.
- Makes adding a new tag-driven system a one-line registration, not a new script.
- Centralizes error handling and debug logging for all tag events.

```
CollectionService (engine)
        │
        ▼
CollectionManager (your registry)
        │
        ├── Tag: "Enemy"      → EnemyComponent.attach / .detach
        ├── Tag: "DamageZone" → DamageZoneComponent.attach / .detach
        └── Tag: "Lootable"   → LootableComponent.attach / .detach
```

---

## 2. COMPONENT CONTRACT

Every module registered into `CollectionManager` **must** expose two functions:

```lua
-- Minimal component interface
export type Component = {
    attach: (instance: Instance) -> (),
    detach: (instance: Instance) -> (),
}
```

- `attach(instance)` — called when an instance receives the tag (or already has it at boot).
- `detach(instance)` — called when the tag is removed OR the instance is destroyed.
- Both functions must be **idempotent-safe**: guard against double-attach or double-detach.
- Neither function should yield; defer async work with `task.spawn`.

---

## 3. COLLECTIONMANAGER MODULE — FULL IMPLEMENTATION

```lua
-- ModuleScript: ServerScriptService/CollectionManager.lua
-- (Mirror to ReplicatedStorage for shared client+server use if needed)

local CollectionService = game:GetService("CollectionService")

type ComponentDef = {
    attach: (instance: Instance) -> (),
    detach: (instance: Instance) -> (),
}

type Registration = {
    tag:        string,
    component:  ComponentDef,
    addedConn:  RBXScriptConnection,
    removedConn: RBXScriptConnection,
    active:     {[Instance]: boolean},
}

local CollectionManager = {}
local registry: {[string]: Registration} = {}

-- ─────────────────────────────────────────────
-- INTERNAL HELPERS
-- ─────────────────────────────────────────────

local function safeAttach(reg: Registration, instance: Instance)
    if reg.active[instance] then
        warn("[CollectionManager] Double-attach blocked:", instance:GetFullName(), "tag:", reg.tag)
        return
    end
    reg.active[instance] = true
    local ok, err = pcall(reg.component.attach, instance)
    if not ok then
        warn("[CollectionManager] attach() error on", instance:GetFullName(), "tag:", reg.tag, "\n", err)
        reg.active[instance] = nil -- rollback so retry is possible
    end
end

local function safeDetach(reg: Registration, instance: Instance)
    if not reg.active[instance] then return end
    reg.active[instance] = nil
    local ok, err = pcall(reg.component.detach, instance)
    if not ok then
        warn("[CollectionManager] detach() error on", instance:GetFullName(), "tag:", reg.tag, "\n", err)
    end
end

-- ─────────────────────────────────────────────
-- PUBLIC API
-- ─────────────────────────────────────────────

--- Register a component module to a tag.
--- Call once per tag, typically from a loader script at game start.
function CollectionManager.register(tag: string, component: ComponentDef)
    assert(type(tag) == "string" and tag ~= "", "[CollectionManager] tag must be a non-empty string")
    assert(type(component.attach) == "function", "[CollectionManager] component.attach must be a function")
    assert(type(component.detach) == "function", "[CollectionManager] component.detach must be a function")

    if registry[tag] then
        warn("[CollectionManager] Tag already registered:", tag, "— skipping duplicate")
        return
    end

    local reg: Registration = {
        tag        = tag,
        component  = component,
        active     = {},
        addedConn  = nil,
        removedConn = nil,
    }

    -- Bootstrap: handle instances already tagged before this call
    for _, inst in CollectionService:GetTagged(tag) do
        safeAttach(reg, inst)
    end

    -- React to future tag additions
    reg.addedConn = CollectionService:GetInstanceAddedSignal(tag):Connect(function(inst)
        safeAttach(reg, inst)
    end)

    -- React to tag removal or instance destruction
    reg.removedConn = CollectionService:GetInstanceRemovedSignal(tag):Connect(function(inst)
        safeDetach(reg, inst)
    end)

    registry[tag] = reg
end

--- Unregister a tag: detach all active instances and disconnect signals.
function CollectionManager.unregister(tag: string)
    local reg = registry[tag]
    if not reg then
        warn("[CollectionManager] Cannot unregister unknown tag:", tag)
        return
    end

    -- Detach all currently active instances
    for inst in reg.active do
        safeDetach(reg, inst)
    end

    reg.addedConn:Disconnect()
    reg.removedConn:Disconnect()
    registry[tag] = nil
end

--- Returns true if a tag is currently registered.
function CollectionManager.isRegistered(tag: string): boolean
    return registry[tag] ~= nil
end

--- Returns how many instances are currently active under a tag.
function CollectionManager.activeCount(tag: string): number
    local reg = registry[tag]
    if not reg then return 0 end
    local count = 0
    for _ in reg.active do count += 1 end
    return count
end

--- Debug: print all registered tags and their active instance counts.
function CollectionManager.debugDump()
    print("── CollectionManager Registry ──")
    for tag, reg in registry do
        local count = 0
        for _ in reg.active do count += 1 end
        print(string.format("  [%s] → %d active instance(s)", tag, count))
    end
    print("────────────────────────────────")
end

return CollectionManager
```

---

## 4. LOADER SCRIPT — WIRING COMPONENTS AT BOOT

```lua
-- Script: ServerScriptService/ComponentLoader.server.lua

local CollectionManager = require(script.Parent.CollectionManager)
local Tags              = require(game.ReplicatedStorage.TagRegistry)

-- Import component modules
local EnemyComponent     = require(script.Parent.Components.EnemyComponent)
local DamageZoneComponent = require(script.Parent.Components.DamageZoneComponent)
local LootableComponent  = require(script.Parent.Components.LootableComponent)

-- One line per system — this is the entire registration surface
CollectionManager.register(Tags.ENEMY,       EnemyComponent)
CollectionManager.register(Tags.DAMAGE_ZONE, DamageZoneComponent)
CollectionManager.register(Tags.LOOTABLE,    LootableComponent)
```

> **Principle**: The loader is the only place that knows which tag maps to which component.
> Components themselves are unaware of their own tag string.

---

## 5. EXAMPLE COMPONENT MODULE

```lua
-- ModuleScript: ServerScriptService/Components/DamageZoneComponent.lua

local Players = game:GetService("Players")

local DamageZoneComponent = {}

-- Per-instance cleanup storage
local connections: {[Instance]: {RBXScriptConnection}} = {}

function DamageZoneComponent.attach(instance: Instance)
    local conns = {}

    local conn = instance.Touched:Connect(function(hit)
        local character = hit.Parent
        local player    = Players:GetPlayerFromCharacter(character)
        if not player then return end

        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid:TakeDamage(10)
        end
    end)

    table.insert(conns, conn)
    connections[instance] = conns
    print("[DamageZone] Attached to", instance:GetFullName())
end

function DamageZoneComponent.detach(instance: Instance)
    local conns = connections[instance]
    if not conns then return end

    for _, c in conns do
        c:Disconnect()
    end
    connections[instance] = nil
    print("[DamageZone] Detached from", instance:GetFullName())
end

return DamageZoneComponent
```

---

## 6. PERFORMANCE PRINCIPLES

| Principle | Rule |
|-----------|------|
| **Single signal per tag** | `CollectionManager` connects once per tag globally, not once per instance |
| **No per-frame GetTagged** | Active set is maintained internally; zero polling |
| **pcall in attach/detach** | One broken component never crashes the entire manager |
| **Lazy component loading** | `require()` component modules only inside `register()` calls, not at module load |
| **Active map is O(1)** | `reg.active[instance]` lookup is constant time |

---

## 7. ARCHITECTURAL PRINCIPLES

1. **One registration per tag.** Registering the same tag twice is blocked with a warning.
2. **Manager owns lifecycle.** External scripts must not directly call `component.attach/detach`.
   All attach/detach flows through `CollectionManager`.
3. **Components are stateless about their tag.** A component module only knows how to set up
   and tear down a single instance. Tag routing is the manager's responsibility.
4. **Bootstrap before signal.** `GetTagged` is always called before `GetInstanceAddedSignal`
   to ensure existing instances are never missed.
5. **Detach is always safe.** `safeDetach` checks `active[instance]` before proceeding —
   destroyed instances will not cause errors if `detach` guards property access.

---

## 8. COMMON ERRORS & SOLUTIONS

### Error 1 — Double registration of the same tag

**Symptom**: `attach()` fires twice for every instance. Connections stack up over time.

**Cause**: `CollectionManager.register("Enemy", ...)` called from two different scripts.

**Fix**: The module blocks this with a `warn` and early return. Audit your loader scripts —
registration must happen in **one place only** (the ComponentLoader).

---

### Error 2 — Component attach errors silently swallowed

**Symptom**: A component never works but no error is thrown in output.

**Cause**: `pcall` in `safeAttach` catches errors and logs them as `warn`, which can be missed.

**Fix**: During development, temporarily replace `pcall` with a direct call to surface the full
stack trace. Switch back to `pcall` for production.

```lua
-- Dev mode: surface full errors
-- reg.component.attach(instance)  ← uncomment temporarily

-- Production mode: safe
local ok, err = pcall(reg.component.attach, instance)
```

---

### Error 3 — detach() accesses destroyed instance properties

**Symptom**: Error: `"attempt to index a nil value"` or `"Instance is not a valid member"` inside `detach`.

**Cause**: `GetInstanceRemovedSignal` fires when instance is destroyed; properties may be gone.

**Fix**: Guard every property access in `detach`.

```lua
function MyComponent.detach(instance: Instance)
    if not instance or not instance.Parent then
        -- Instance is destroyed; only do memory cleanup, no property access
        connections[instance] = nil
        return
    end
    -- Safe to access instance here
end
```

---

### Error 4 — register() called before DataModel is ready

**Symptom**: `GetTagged` returns empty even though tagged instances exist in the place file.

**Cause**: `register()` is called before the game world finishes loading (e.g., in a `ModuleScript`
that runs at require-time rather than inside a Script that waits for game load).

**Fix**: Call `register()` inside a server `Script`, not at module-load time of a `ModuleScript`.
If needed, gate with `game:IsLoaded()` or `game.Loaded:Wait()`.

```lua
-- server Script
if not game:IsLoaded() then game.Loaded:Wait() end
CollectionManager.register(Tags.ENEMY, EnemyComponent)
```

---

### Error 5 — unregister() not called on game shutdown / round reset

**Symptom**: After a round reset, components re-attach on top of existing state causing duplicate effects.

**Cause**: Old registrations are never torn down before new ones are made.

**Fix**: Call `CollectionManager.unregister(tag)` for all tags before reinitializing systems on round reset.

```lua
local function onRoundEnd()
    CollectionManager.unregister(Tags.ENEMY)
    CollectionManager.unregister(Tags.DAMAGE_ZONE)
    -- ... then re-register for next round
end
```

---

### Error 6 — Component uses upvalue state shared across instances

**Symptom**: Damaging one enemy damages all enemies. State from one instance leaks into another.

**Cause**: Component stores per-instance data in a module-level variable rather than keyed by instance.

**Fix**: Always key per-instance state by `instance` reference.

```lua
-- BAD
local health = 100 -- shared across all instances

-- GOOD
local healthMap: {[Instance]: number} = {}

function MyComponent.attach(instance: Instance)
    healthMap[instance] = 100
end

function MyComponent.detach(instance: Instance)
    healthMap[instance] = nil
end
```

---

## 9. CLIENT-SIDE COLLECTIONMANAGER

The same `CollectionManager` module works on the client. Place it in `ReplicatedStorage`.

**Client rules**:
- Only react to tags that the server replicates (server-added tags).
- Client components handle VFX, UI, audio — never authoritative gameplay state.
- Never call `CollectionService:AddTag` from the client for server-critical tags.

```lua
-- LocalScript: StarterPlayerScripts/ClientComponentLoader.lua
local CollectionManager = require(game.ReplicatedStorage.CollectionManager)
local Tags              = require(game.ReplicatedStorage.TagRegistry)
local EnemyMarkerUI     = require(game.ReplicatedStorage.Components.EnemyMarkerUI)

CollectionManager.register(Tags.ENEMY, EnemyMarkerUI)
```

---

## 10. DEBUG & VALIDATION UTILITIES

```lua
-- Print full registry state at any point
CollectionManager.debugDump()
-- Output:
-- ── CollectionManager Registry ──
--   [Enemy]      → 12 active instance(s)
--   [DamageZone] → 3 active instance(s)
--   [Lootable]   → 7 active instance(s)
-- ────────────────────────────────

-- Check count programmatically
local enemyCount = CollectionManager.activeCount(Tags.ENEMY)
print("Live enemies:", enemyCount)

-- Check if a tag is currently managed
if not CollectionManager.isRegistered(Tags.BOSS) then
    warn("BossComponent was never registered!")
end
```

---

## 11. QUICK REFERENCE

```
CollectionManager.register(tag, component)   → register a tag → component mapping
CollectionManager.unregister(tag)            → teardown all instances + disconnect signals
CollectionManager.isRegistered(tag)          → boolean
CollectionManager.activeCount(tag)           → number of live instances under tag
CollectionManager.debugDump()                → print full registry to output
```

**Component contract**:
```
component.attach(instance: Instance) -> ()   -- setup; must not yield
component.detach(instance: Instance) -> ()   -- teardown; must not yield; guard destroyed state
```

**Golden rules**:
- Register each tag exactly once, in one loader script.
- Components must key all per-instance state by `instance` reference.
- Always guard property access in `detach()` against destroyed instances.
- Use `pcall` in production; remove it temporarily to debug attach/detach errors.
- Call `unregister()` before reinitializing systems on round/map reset.