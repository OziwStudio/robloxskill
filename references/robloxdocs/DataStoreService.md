# DataStoreService
> Roblox Luau reference for persistence, data integrity, error handling, and transactional logic.
> Use this file as the single DataStoreService guide for AI reasoning.

## 1. Purpose

DataStoreService is the engine service for persisting game state between sessions.
It is the backbone of economy, progression, and save systems.

Use it when you want:
- player inventory persistence
- high-score tracking
- global settings storage
- cross-server data synchronization

Do not use it as a message bus, a realtime database for high-frequency updates, or a replacement for security validation.

## 2. Core Principles

1. Persistence is asynchronous and may fail.
2. `UpdateAsync` is the safe default for critical data.
3. Every read and write must be wrapped in `pcall`.
4. Server authority is absolute.
5. Throttling is real and must be respected.
6. Data must be serializable.
7. Session locking prevents multi-server overwrite conflicts.
8. Local state is temporary; persisted state is the source of truth.

## 3. API Surface

### 3.1 GetDataStore

`DataStoreService:GetDataStore(name, [scope])`

- Returns a DataStore object.
- Use scopes only when they reduce confusion.
- Prefer simple naming when possible.

### 3.2 GetAsync

`DataStore:GetAsync(key)`

- Returns the current value.
- Returns `nil` if the key does not exist.
- Treat it as a read operation that may yield or fail.

### 3.3 SetAsync

`DataStore:SetAsync(key, value)`

- Overwrites the value completely.
- Use only when race conditions are not a problem.
- Do not use for critical cumulative data.

### 3.4 UpdateAsync

`DataStore:UpdateAsync(key, transformFunction)`

- Reads, transforms, and writes in one atomic transaction.
- Use for currency, inventory, progression, and any state that must not be lost.
- This is the default choice for important data.

### 3.5 RemoveAsync

`DataStore:RemoveAsync(key)`

- Deletes the key.
- Use with caution and only when deletion is intended.

## 4. Replication Rules

| Action | Server | Client |
|---|---|---|
| GetAsync | allowed | forbidden |
| UpdateAsync | allowed | forbidden |
| SetAsync | allowed | forbidden |
| RemoveAsync | allowed | forbidden |

Rules:
- Never expose DataStore calls to the client.
- Client data requests should use remotes or replicated state.
- Client claims about money, inventory, or unlocks must never be trusted.

## 5. Golden Rule

`UpdateAsync` is the correct default for critical persistence.

### Bad

```lua
local data = ds:GetAsync(key)
data.Coins += 10
ds:SetAsync(key, data)
```

### Good

```lua
ds:UpdateAsync(key, function(oldData)
    local data = oldData or {Coins = 0}
    data.Coins += 10
    return data
end)
```

## 6. Error Handling

DataStore calls may fail.
Wrap every call in `pcall`, and plan a fallback path.

```lua
local success, result = pcall(function()
    return myStore:GetAsync(userId)
end)

if not success then
    warn("DataStore error:", result)
    return
end
```

## 7. Throttling and Budgeting

Roblox limits request budgets.

Rules:
- do not save every tiny change immediately
- cache data in memory
- autosave at intervals
- save on leave
- spread writes across time when many players are active

## 8. Session Locking

Session locking prevents two servers from writing conflicting values for the same player.

Rules:
- assign a session identity on join
- verify the lock before writing
- release the lock on leave
- reject stale writes from other servers

This helps prevent rollback and duplication problems.

## 9. Serialization Rules

DataStores only accept serializable data.

Allowed:
- string
- number
- boolean
- table

Forbidden:
- Instances
- functions
- threads
- userdata
- metatables

Rules:
- convert Instances into ID strings
- reconstruct runtime objects after load
- keep schema shape predictable

## 10. Data Migration

When schema changes, migrate old data safely.

Rules:
- detect the old format
- transform it to the new format
- save the new format back
- keep backward compatibility for existing players

## 11. Performance Rules

- read once, cache locally
- write only when needed
- avoid frequent live reads
- use `task.spawn` or `task.defer` for non-blocking save flow
- keep server-side player cache in memory

## 12. Security Rules

Never trust the client.

Rules:
- validate all values before saving
- reject impossible values
- treat DataStore as backup, not live gameplay authority
- server decides what is valid

## 13. Common Failures

### 13.1 Missing pcall

Problem: temporary outage crashes the script.
Fix: wrap every call.

### 13.2 SetAsync overwrite

Problem: data rolls back or gets replaced.
Fix: use `UpdateAsync`.

### 13.3 Cache leak

Problem: local cache never clears on leave.
Fix: remove player entries on `PlayerRemoving`.

### 13.4 Throttling

Problem: too many saves at once.
Fix: spread writes and batch updates.

## 14. Professional Behavior Rules for AI

- prioritize data integrity over brevity
- default to `UpdateAsync`
- warn about budgets and throttling
- use caching to reduce API calls
- treat player progress as critical
- design for failure, not hope

## 15. Minimal Checklist

- every call uses `pcall`
- no critical `SetAsync`
- autosave exists
- final save on leave exists
- data is serializable
- migration path is planned
- budget handling is present

## 16. Final Rule

DataStoreService should be invisible to the player.
If they notice it, something went wrong.

