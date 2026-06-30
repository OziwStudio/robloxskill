# ReplicationEngine — CollectionService Reference

> **Scope:** CollectionService only. Server-authoritative, tag-driven instance management in Roblox Luau.  
> **Audience:** Principal engineer. Attach to any AI conversation for accurate, production-grade output.

---

## 1. What CollectionService Is

`CollectionService` is a Roblox engine service that lets you **tag instances with string labels** and observe those tags engine-wide. It is the canonical replacement for folder-based grouping, `IsA` loops, and manual instance registries.

Tags persist across `Clone`, survive `Instance:Destroy` signal order, and replicate from server to all clients automatically when set on the server.

---

## 2. Core API Surface

| Method | Boundary | Returns |
|---|---|---|
| `CollectionService:AddTag(instance, tag)` | Server / Client | `void` |
| `CollectionService:RemoveTag(instance, tag)` | Server / Client | `void` |
| `CollectionService:HasTag(instance, tag)` | Both | `boolean` |
| `CollectionService:GetTagged(tag)` | Both | `{Instance}` |
| `CollectionService:GetTags(instance)` | Both | `{string}` |
| `CollectionService:GetInstanceAddedSignal(tag)` | Both | `RBXScriptSignal` |
| `CollectionService:GetInstanceRemovedSignal(tag)` | Both | `RBXScriptSignal` |

**Replication rule:** Tags added server-side replicate to clients. Tags added client-side are **local only** and never reach the server or other clients.

---

## 3. Principal Architecture

```
Server
  └─ CollectionService (source of truth)
       ├─ AddTag / RemoveTag  →  replicates to all clients
       └─ GetInstanceAddedSignal / GetInstanceRemovedSignal

Client
  └─ CollectionService (read replica)
       ├─ GetTagged (returns server-replicated + local tags)
       └─ GetInstanceAddedSignal fires when server replica arrives
```

Never use `GetTagged` on the client as a reliable snapshot at game start — tagged instances may not have replicated yet. Always use signals.

---

## 4. Correct Initialization Pattern

```lua
local CollectionService = game:GetService("CollectionService")

local TAG = "Interactable"

local function onAdded(instance: Instance)
    -- setup logic
end

local function onRemoved(instance: Instance)
    -- teardown logic
end

for _, instance in CollectionService:GetTagged(TAG) do
    task.spawn(onAdded, instance)
end

CollectionService:GetInstanceAddedSignal(TAG):Connect(onAdded)
CollectionService:GetInstanceRemovedSignal(TAG):Connect(onRemoved)
```

`task.spawn` prevents one failing handler from blocking the bootstrap loop.

---

## 5. Multi-Tag Pattern (Entity Component)

```lua
local CollectionService = game:GetService("CollectionService")

local function hasAll(instance: Instance, tags: {string}): boolean
    for _, tag in tags do
        if not CollectionService:HasTag(instance, tag) then
            return false
        end
    end
    return true
end

local function onEnemyAdded(instance: Instance)
    if not hasAll(instance, {"Enemy", "Damageable"}) then return end
    -- register entity
end

CollectionService:GetInstanceAddedSignal("Enemy"):Connect(onEnemyAdded)
CollectionService:GetInstanceAddedSignal("Damageable"):Connect(onEnemyAdded)
```

---

## 6. Registry Pattern (O(1) Lookup)

```lua
local CollectionService = game:GetService("CollectionService")

local TAG = "Zone"
local registry: {[Instance]: true} = {}

CollectionService:GetInstanceAddedSignal(TAG):Connect(function(inst)
    registry[inst] = true
end)

CollectionService:GetInstanceRemovedSignal(TAG):Connect(function(inst)
    registry[inst] = nil
end)

for _, inst in CollectionService:GetTagged(TAG) do
    registry[inst] = true
end

local function getAllZones(): {Instance}
    local result = {}
    for inst in registry do
        table.insert(result, inst)
    end
    return result
end
```

Avoids repeated `GetTagged` calls (iterates entire tagged list every call).

---

## 7. Cleanup Pattern (Maid / Connection Table)

```lua
local CollectionService = game:GetService("CollectionService")

local TAG = "Trap"
local connections: {[Instance]: {RBXScriptConnection}} = {}

local function onAdded(trap: Instance)
    local part = trap:FindFirstChild("Hitbox") :: BasePart
    if not part then return end

    local conns = {}
    conns[1] = part.Touched:Connect(function(hit)
        -- damage logic
    end)
    connections[trap] = conns
end

local function onRemoved(trap: Instance)
    local conns = connections[trap]
    if conns then
        for _, c in conns do c:Disconnect() end
        connections[trap] = nil
    end
end

for _, trap in CollectionService:GetTagged(TAG) do
    task.spawn(onAdded, trap)
end

CollectionService:GetInstanceAddedSignal(TAG):Connect(onAdded)
CollectionService:GetInstanceRemovedSignal(TAG):Connect(onRemoved)
```

---

## 8. Dynamic Tagging (Runtime)

```lua
local CollectionService = game:GetService("CollectionService")

local function markAsCollected(item: Instance)
    CollectionService:RemoveTag(item, "Collectible")
    CollectionService:AddTag(item, "Collected")
end
```

Tag swaps are atomic from the signal consumer's perspective. `Removed` fires before `Added` on the same frame when done in sequence.

---

## 9. Replication Timing — Critical Rules

| Situation | Behavior |
|---|---|
| Server adds tag to a replicated instance | Clients receive `GetInstanceAddedSignal` after replication lag |
| Server destroys instance before client replicates tag | Client may never fire `Added` |
| Client adds tag locally | No server or peer replication |
| StreamingEnabled + instance not streamed yet | `GetTagged` will not include it on client |

**Never assume** `GetTagged` on client equals server state at any given moment.

---

## 10. StreamingEnabled Compatibility

When `StreamingEnabled` is on, instances stream in/out. CollectionService signals fire correctly as instances enter/leave the client's stream bubble — treat this as normal `Added/Removed` lifecycle. No special handling required beyond the standard signal pattern.

---

## 11. Common Errors & Fixes

### E1 — Handler runs before instance is fully loaded

```
-- symptom: FindFirstChild returns nil inside onAdded
-- cause:   instance exists but children not yet replicated
```

```lua
local function onAdded(instance: Instance)
    if not instance:IsDescendantOf(game) then return end
    instance.DescendantAdded:Wait()  -- or use a specific child wait
    -- safe to access children now
end
```

Better: use `WaitForChild` with a timeout or restructure initialization to be event-driven on child presence.

---

### E2 — Duplicate signal connections on re-tag

```
-- symptom: handler fires multiple times per event
-- cause:   AddTag called again on already-tagged instance
--          CollectionService does NOT deduplicate — signal fires once per AddTag call
```

```lua
local function safeTag(instance: Instance, tag: string)
    if not CollectionService:HasTag(instance, tag) then
        CollectionService:AddTag(instance, tag)
    end
end
```

---

### E3 — Memory leak from orphaned connections

```
-- symptom: connections table grows unbounded
-- cause:   instance destroyed without RemoveTag being called
--          GetInstanceRemovedSignal still fires on Destroy, so cleanup runs
--          BUT if you disconnect the Removed signal too early you miss it
```

Rule: **never disconnect `GetInstanceRemovedSignal` before the removed handler cleans up connections**.

---

### E4 — GetTagged used in hot loop

```
-- symptom: frame drops on large worlds
-- cause:   GetTagged allocates a new table each call
```

Use the registry pattern (Section 6) to cache the set and index it in O(1).

---

### E5 — Client-side tag not visible to server

```
-- symptom: server logic never triggers for client-tagged instance
-- cause:   client tag is local-only
```

Fix: fire a `RemoteEvent` to the server and tag there.

---

## 12. Tag Naming Convention

```
Domain_Entity_State
Examples:
  "Combat_Enemy_Active"
  "World_Prop_Destructible"
  "UI_Button_Highlighted"
```

Flat lowercase tags (`"enemy"`) cause collisions across systems at scale. Use namespaced strings.

---

## 13. What CollectionService Is NOT

- Not a DataStore. Tags do not persist across server sessions.
- Not a messaging bus. Use `BindableEvent` / `RemoteEvent` for events.
- Not a component system by itself. Build one on top of it (see Section 5).
- Not safe as a solo authority on client. Always verify tag intent server-side for security-sensitive logic.

---

## 14. Security Principle

Never trust client-reported tag state for any game-economy, anti-cheat, or damage decision. The server is the only authoritative source. Clients can read replicated tags for visual/UX purposes only.

---

*End of ReplicationEngine.md — CollectionService scope only.*