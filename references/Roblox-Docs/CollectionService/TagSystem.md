# TagSystem — CollectionService Reference
> Part of: `CollectionService` · Roblox Luau · Production Reference
> Scope: Tag management, instance querying, signal handling, lifecycle, anti-patterns, error catalog

---

## 1. SERVICE OVERVIEW

`CollectionService` is a Roblox engine service that allows tagging any `Instance` with one or more
string labels. Tags persist across script boundaries, replicate from server to client, and survive
re-parenting. The service is the canonical way to build **data-driven, decoupled systems** where
logic is driven by tag presence rather than class hierarchy or naming conventions.

```lua
-- Canonical access pattern
local CollectionService = game:GetService("CollectionService")
```

Tags are **strings**. They are case-sensitive. `"Enemy"` ≠ `"enemy"`.

---

## 2. CORE API — COMPLETE REFERENCE

### 2.1 AddTag

```lua
CollectionService:AddTag(instance: Instance, tag: string): void
```

- Attaches `tag` to `instance`. No-op if the tag already exists on that instance.
- Safe to call multiple times; does not duplicate.
- Server → replicates to all clients automatically.
- Client → local-only, never replicates to server.

```lua
-- Server script
local part = workspace.RedZone
CollectionService:AddTag(part, "DamageZone")
CollectionService:AddTag(part, "NoRespawn")
```

---

### 2.2 RemoveTag

```lua
CollectionService:RemoveTag(instance: Instance, tag: string): void
```

- Detaches `tag` from `instance`. No-op if tag does not exist.
- Triggers `GetInstanceRemovedSignal` for that tag.

```lua
CollectionService:RemoveTag(part, "DamageZone")
```

---

### 2.3 HasTag

```lua
CollectionService:HasTag(instance: Instance, tag: string): boolean
```

- Returns `true` if `instance` currently carries `tag`.
- O(1) lookup. Safe to call every frame if needed.

```lua
if CollectionService:HasTag(hit, "Enemy") then
    hit:FindFirstChild("Humanoid"):TakeDamage(10)
end
```

---

### 2.4 GetTags

```lua
CollectionService:GetTags(instance: Instance): {string}
```

- Returns a **snapshot** array of all tags on `instance` at call time.
- Mutating the returned table has no effect on the instance.

```lua
local tags = CollectionService:GetTags(workspace.Chest)
for _, tag in tags do
    print(tag) -- "Interactable", "Lootable", etc.
end
```

---

### 2.5 GetTagged

```lua
CollectionService:GetTagged(tag: string): {Instance}
```

- Returns a **snapshot** array of every instance currently carrying `tag`.
- Array order is **not** guaranteed to be stable across calls.
- Snapshot: instances added/removed after the call are not reflected.

```lua
-- Iterate all enemies at this moment
for _, enemy in CollectionService:GetTagged("Enemy") do
    -- operate on enemy
end
```

> **Performance principle**: Cache the result when iterating a large tag set in one frame.
> Do not call `GetTagged` inside a tight loop (e.g., `RunService.Heartbeat`) without throttling.

---

### 2.6 GetInstanceAddedSignal

```lua
CollectionService:GetInstanceAddedSignal(tag: string): RBXScriptSignal
```

- Fires whenever an instance **receives** `tag`, including instances already tagged at connect time
  that are later parented into the DataModel.
- Does **not** fire retroactively for instances that already have the tag when you connect.
  Use `GetTagged` + `GetInstanceAddedSignal` together for complete coverage.

```lua
-- Canonical pattern: handle existing + future tagged instances
local TAG = "Interactable"

local function onTagAdded(instance: Instance)
    -- setup logic
    print(instance.Name, "is now Interactable")
end

-- Handle already-existing instances
for _, inst in CollectionService:GetTagged(TAG) do
    onTagAdded(inst)
end

-- Handle future instances
CollectionService:GetInstanceAddedSignal(TAG):Connect(onTagAdded)
```

---

### 2.7 GetInstanceRemovedSignal

```lua
CollectionService:GetInstanceRemovedSignal(tag: string): RBXScriptSignal
```

- Fires when a tag is **removed** from an instance, OR when the instance is **destroyed**.
- Use this to tear down connections, clean up UI, or cancel tasks tied to that instance.

```lua
local function onTagRemoved(instance: Instance)
    print(instance.Name, "lost Interactable tag or was destroyed")
    -- cleanup
end

CollectionService:GetInstanceRemovedSignal(TAG):Connect(onTagRemoved)
```

---

## 3. REPLICATION RULES

| Action            | Server → Client | Client → Server |
|-------------------|-----------------|-----------------|
| AddTag            | ✅ Replicates   | ❌ Local only   |
| RemoveTag         | ✅ Replicates   | ❌ Local only   |
| HasTag (read)     | ✅ Reads server state | ✅ Reads local state |
| GetTagged (read)  | ✅ Server snapshot | ✅ Local snapshot |

**Principle**: Only the server should mutate tags for game-critical state.
Client-side tags are valid for local UI state, local VFX, or client-predicted states.

---

## 4. LIFECYCLE MANAGEMENT — MAID / JANITOR PATTERN

Tag signals must be cleaned up to prevent memory leaks and ghost callbacks.

```lua
-- Using a simple cleanup table (no external dependency)
local connections: {RBXScriptConnection} = {}

local function setup(instance: Instance)
    local conn = instance.AncestryChanged:Connect(function()
        -- react to re-parent
    end)
    table.insert(connections, conn)
end

local function teardown()
    for _, conn in connections do
        conn:Disconnect()
    end
    table.clear(connections)
end

CollectionService:GetInstanceAddedSignal("Enemy"):Connect(setup)
CollectionService:GetInstanceRemovedSignal("Enemy"):Connect(function(inst)
    teardown()
end)
```

**Rule**: Every `GetInstanceAddedSignal` connection that creates inner connections
**must** pair with a `GetInstanceRemovedSignal` that cleans them up.

---

## 5. MULTI-TAG QUERYING

CollectionService has no native AND/OR multi-tag filter. Implement it manually.

```lua
-- AND filter: instance must have ALL of these tags
local function hasAllTags(instance: Instance, tags: {string}): boolean
    for _, tag in tags do
        if not CollectionService:HasTag(instance, tag) then
            return false
        end
    end
    return true
end

-- OR filter: instance must have AT LEAST ONE tag
local function hasAnyTag(instance: Instance, tags: {string}): boolean
    for _, tag in tags do
        if CollectionService:HasTag(instance, tag) then
            return true
        end
    end
    return false
end

-- Usage: find all instances tagged both "Enemy" and "Elite"
local eliteEnemies = {}
for _, inst in CollectionService:GetTagged("Enemy") do
    if hasAllTags(inst, {"Enemy", "Elite"}) do
        table.insert(eliteEnemies, inst)
    end
end
```

---

## 6. TAG CONSTANTS — CENTRALIZED REGISTRY PATTERN

**Never hardcode tag strings inline.** Use a centralized constants module.

```lua
-- ModuleScript: ReplicatedStorage/TagRegistry.lua
local TagRegistry = {
    -- Combat
    ENEMY           = "Enemy",
    ELITE_ENEMY     = "Elite",
    BOSS            = "Boss",
    DAMAGE_ZONE     = "DamageZone",
    SAFE_ZONE       = "SafeZone",

    -- Interaction
    INTERACTABLE    = "Interactable",
    LOOTABLE        = "Lootable",
    DOOR            = "Door",

    -- State flags
    RESPAWNING      = "Respawning",
    INVINCIBLE      = "Invincible",
    STUNNED         = "Stunned",

    -- System
    MANAGED         = "Managed",
    INITIALIZED     = "Initialized",
}

return TagRegistry
```

```lua
-- Consumer script
local Tags = require(ReplicatedStorage.TagRegistry)
local CS   = game:GetService("CollectionService")

CS:AddTag(npcModel, Tags.ENEMY)
CS:AddTag(npcModel, Tags.ELITE_ENEMY)

if CS:HasTag(target, Tags.INVINCIBLE) then return end
```

---

## 7. TAG-DRIVEN COMPONENT SYSTEM (ADVANCED)

Tags are the backbone of ECS-like component architectures in Roblox.

```lua
-- ComponentBase pattern
-- Each component module self-registers via CollectionService signals

-- ModuleScript: ServerScriptService/Components/HealthBar.lua
local CollectionService = game:GetService("CollectionService")
local TAG = "HasHealthBar"

local HealthBar = {}
local activeComponents: {[Instance]: any} = {}

local function attach(instance: Instance)
    if activeComponents[instance] then return end
    local component = {
        instance = instance,
        -- ... init UI, connections, etc.
    }
    activeComponents[instance] = component
    print("[HealthBar] Attached to", instance.Name)
end

local function detach(instance: Instance)
    local component = activeComponents[instance]
    if not component then return end
    -- cleanup
    activeComponents[instance] = nil
    print("[HealthBar] Detached from", instance.Name)
end

-- Bootstrap
for _, inst in CollectionService:GetTagged(TAG) do
    attach(inst)
end
CollectionService:GetInstanceAddedSignal(TAG):Connect(attach)
CollectionService:GetInstanceRemovedSignal(TAG):Connect(detach)

return HealthBar
```

---

## 8. PERFORMANCE PRINCIPLES

| Principle | Rule |
|-----------|------|
| **HasTag is O(1)** | Safe inside `Heartbeat` for single instance checks |
| **GetTagged is O(n)** | Cache result; do not call per-frame on large tag sets |
| **Signal > Poll** | Prefer `GetInstanceAddedSignal` over repeated `GetTagged` scans |
| **Avoid per-frame GetTagged** | Throttle to max once per 0.1s or use signals instead |
| **Snapshot immutability** | Never store `GetTagged` result and expect it to update |

```lua
-- BAD: polling GetTagged every frame
RunService.Heartbeat:Connect(function()
    for _, enemy in CollectionService:GetTagged("Enemy") do -- O(n) every frame
        updateEnemy(enemy)
    end
end)

-- GOOD: maintain your own live set via signals
local liveEnemies: {[Instance]: boolean} = {}

for _, e in CollectionService:GetTagged("Enemy") do liveEnemies[e] = true end
CollectionService:GetInstanceAddedSignal("Enemy"):Connect(function(e) liveEnemies[e] = true end)
CollectionService:GetInstanceRemovedSignal("Enemy"):Connect(function(e) liveEnemies[e] = nil end)

RunService.Heartbeat:Connect(function()
    for enemy in liveEnemies do
        updateEnemy(enemy) -- operates on always-current set, zero GetTagged calls
    end
end)
```

---

## 9. COMMON ERRORS & SOLUTIONS

### Error 1 — Missing retroactive handling

**Symptom**: Component logic never runs on instances that already existed when the script started.

**Cause**: Only connected to `GetInstanceAddedSignal`, skipping the `GetTagged` bootstrap.

**Fix**:
```lua
-- Always bootstrap FIRST, then connect signal
for _, inst in CollectionService:GetTagged(TAG) do onAdded(inst) end
CollectionService:GetInstanceAddedSignal(TAG):Connect(onAdded)
```

---

### Error 2 — Tag case mismatch

**Symptom**: `HasTag` returns `false` even though the tag appears in Studio's Tag Editor.

**Cause**: `"enemy"` ≠ `"Enemy"`. Tags are case-sensitive.

**Fix**: Always use the `TagRegistry` constants module. Never type tag strings inline.

---

### Error 3 — Client adding tags expecting server reaction

**Symptom**: Server-side logic never fires after client calls `AddTag`.

**Cause**: Client tags do not replicate to server.

**Fix**: Client fires a `RemoteEvent`; server validates and calls `AddTag` server-side.

```lua
-- Client
TagRemote:FireServer("DamageZone", workspace.Part)

-- Server
TagRemote.OnServerEvent:Connect(function(player, tagName, instance)
    -- validate player permission, tag name whitelist, instance ownership
    if allowedTags[tagName] and isOwned(player, instance) then
        CollectionService:AddTag(instance, tagName)
    end
end)
```

---

### Error 4 — Memory leak from uncleared signal connections

**Symptom**: Scripts accumulate thousands of live connections over a session; performance degrades.

**Cause**: `GetInstanceAddedSignal` handler creates inner connections that are never disconnected
on `GetInstanceRemovedSignal`.

**Fix**: Store inner connections per-instance in a table; disconnect inside the removed signal handler.

```lua
local innerConns: {[Instance]: {RBXScriptConnection}} = {}

CollectionService:GetInstanceAddedSignal(TAG):Connect(function(inst)
    innerConns[inst] = {
        inst.Touched:Connect(function() end),
        inst.AncestryChanged:Connect(function() end),
    }
end)

CollectionService:GetInstanceRemovedSignal(TAG):Connect(function(inst)
    if innerConns[inst] then
        for _, c in innerConns[inst] do c:Disconnect() end
        innerConns[inst] = nil
    end
end)
```

---

### Error 5 — GetTagged snapshot treated as live

**Symptom**: Script misses newly tagged instances or references destroyed ones.

**Cause**: Developer stores `GetTagged` result and iterates it minutes later.

**Fix**: Always treat `GetTagged` as a one-time snapshot. Use the live-set pattern (Section 8).

---

### Error 6 — Tagging during destroy causes signal race

**Symptom**: `GetInstanceRemovedSignal` fires, but attempting to access properties on the instance
inside the handler throws "Instance is not a valid member".

**Cause**: The instance is mid-destroy when the signal fires.

**Fix**: Guard all property access inside removed signal handlers.

```lua
CollectionService:GetInstanceRemovedSignal(TAG):Connect(function(inst)
    if not inst or not inst.Parent then return end -- guard
    -- safe to access inst here
end)
```

---

## 10. STUDIO INTEGRATION

Tags can be added visually in Studio via **Tag Editor** (Plugin tab or `View > Tag Editor`).
Tags set in Studio are saved in the place file and are present at runtime immediately.

- Use Tag Editor during level design to mark gameplay zones, props, NPCs.
- Use `CollectionService` in scripts to react to those tags at runtime.
- Tags survive `Clone()` — a cloned instance retains all tags of its source.

```lua
-- Tags survive Clone
local original = workspace.EnemyTemplate -- has tag "Enemy"
local clone = original:Clone()
print(CollectionService:HasTag(clone, "Enemy")) -- true
clone.Parent = workspace
```

---

## 11. TESTING & VALIDATION UTILITIES

```lua
-- Debug: print all tags on every instance in a model
local function auditTags(model: Model)
    for _, inst in model:GetDescendants() do
        local tags = CollectionService:GetTags(inst)
        if #tags > 0 then
            print(inst:GetFullName(), "→", table.concat(tags, ", "))
        end
    end
end

auditTags(workspace)
```

```lua
-- Assert a tag exists (for test scripts)
local function assertTagged(inst: Instance, tag: string)
    assert(
        CollectionService:HasTag(inst, tag),
        string.format("[TagSystem] Expected tag '%s' on '%s'", tag, inst:GetFullName())
    )
end
```

---

## 12. SECURITY RULES

1. **Never trust client-sent tag names.** Whitelist all valid tags server-side.
2. **Never allow client to add/remove gameplay-critical tags** (health flags, admin flags, etc.)
3. **Validate instance ownership** before server applies a tag on behalf of a client request.
4. **Audit tag removal** — a malicious client triggering tag removal via exploit cannot replicate to server, but if you have any RemoteEvent that removes tags, guard it tightly.

---

## 13. QUICK REFERENCE CHEATSHEET

```
CollectionService:AddTag(inst, tag)              → attach tag
CollectionService:RemoveTag(inst, tag)           → detach tag
CollectionService:HasTag(inst, tag)              → boolean check
CollectionService:GetTags(inst)                  → snapshot of all tags on inst
CollectionService:GetTagged(tag)                 → snapshot of all inst with tag
CollectionService:GetInstanceAddedSignal(tag)    → signal: fired when inst receives tag
CollectionService:GetInstanceRemovedSignal(tag)  → signal: fired when inst loses tag / destroyed
```

**Golden rules**:
- Always bootstrap with `GetTagged` before connecting `GetInstanceAddedSignal`.
- Always clean up inner connections inside `GetInstanceRemovedSignal`.
- Always centralize tag strings in a `TagRegistry` module.
- Server owns tag mutations for all gameplay-critical state.
- Treat every `GetTagged` result as a snapshot, not a live view.