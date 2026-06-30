# EventSystem — CollectionService Reference
> Part of: `CollectionService` · Roblox Luau · Production Reference
> Scope: Tag-driven event bus, signal lifecycle, cross-system communication via tags, cleanup, error catalog

---

## 1. WHAT IS EVENTSYSTEM

`EventSystem` is a **custom event bus** built to work alongside `CollectionService`.
It is not a Roblox built-in — it is an architectural pattern you implement.

**Purpose**: Decouple systems so they communicate through named events rather than direct
module references. One system fires an event by name; any number of other systems listen
to that name without knowing who fired it.

**Why pair it with CollectionService**:
- Tags describe *what* an instance is. Events describe *what happened* to it.
- Tag signals (`GetInstanceAddedSignal`) handle structural changes (instance gains/loses a tag).
- EventSystem handles **gameplay signals** between systems: damage dealt, item picked up, round ended.
- Together they form a complete decoupled architecture: tags route instances to components;
  events route actions between systems.

```
System A (fires)           EventSystem            System B (listens)
─────────────              ───────────            ──────────────────
EnemyDied:Fire(data)  →  "EnemyDied" bus  →  ScoreSystem:onEnemyDied(data)
                                           →  AudioSystem:onEnemyDied(data)
                                           →  QuestSystem:onEnemyDied(data)
```

---

## 2. CORE DESIGN PRINCIPLES

1. **No direct module coupling.** Systems never `require()` each other to call functions.
   All cross-system calls go through EventSystem.
2. **Named events are strings.** Use a centralized `EventRegistry` constants module — never
   inline string literals in fire/listen calls.
3. **Events are fire-and-forget.** The firing system does not know or care who is listening.
4. **Listeners are responsible for their own cleanup.** Every `listen()` call returns a
   disconnect handle; callers must store and disconnect it on teardown.
5. **EventSystem does not own game state.** It is a pure signal router — no data is stored
   inside the bus between events.
6. **Events fired from server stay on server; events fired from client stay on client.**
   Cross-boundary communication still requires `RemoteEvent` / `RemoteFunction`.

---

## 3. EVENT REGISTRY — CENTRALIZED CONSTANTS

```lua
-- ModuleScript: ReplicatedStorage/EventRegistry.lua

local EventRegistry = {
    -- Combat
    ENEMY_DIED          = "EnemyDied",
    PLAYER_DAMAGED      = "PlayerDamaged",
    PLAYER_DIED         = "PlayerDied",
    BOSS_SPAWNED        = "BossSpawned",

    -- Interaction
    ITEM_COLLECTED      = "ItemCollected",
    DOOR_OPENED         = "DoorOpened",
    ZONE_ENTERED        = "ZoneEntered",
    ZONE_EXITED         = "ZoneExited",

    -- Round lifecycle
    ROUND_STARTED       = "RoundStarted",
    ROUND_ENDED         = "RoundEnded",
    ROUND_COUNTDOWN     = "RoundCountdown",

    -- CollectionService integration
    TAG_COMPONENT_READY = "TagComponentReady",
    TAG_COMPONENT_TORN  = "TagComponentTorn",
}

return EventRegistry
```

> **Rule**: Add new events here first, then use `EventRegistry.EVENT_NAME` everywhere.
> Never type event name strings directly in scripts.

---

## 4. EVENTSYSTEM MODULE — FULL IMPLEMENTATION

```lua
-- ModuleScript: ReplicatedStorage/EventSystem.lua

export type Listener = (data: any) -> ()
export type Unsubscribe = () -> ()

type EventEntry = {
    listeners: {[number]: Listener},
    nextId:    number,
}

local EventSystem = {}
local bus: {[string]: EventEntry} = {}

-- ─────────────────────────────────────────────
-- INTERNAL
-- ─────────────────────────────────────────────

local function getOrCreate(eventName: string): EventEntry
    if not bus[eventName] then
        bus[eventName] = { listeners = {}, nextId = 1 }
    end
    return bus[eventName]
end

-- ─────────────────────────────────────────────
-- PUBLIC API
-- ─────────────────────────────────────────────

--- Subscribe to a named event.
--- Returns an unsubscribe function — call it to stop listening.
function EventSystem.listen(eventName: string, listener: Listener): Unsubscribe
    assert(type(eventName) == "string" and eventName ~= "",
        "[EventSystem] eventName must be a non-empty string")
    assert(type(listener) == "function",
        "[EventSystem] listener must be a function")

    local entry = getOrCreate(eventName)
    local id    = entry.nextId
    entry.nextId += 1
    entry.listeners[id] = listener

    -- Return unsubscribe handle
    return function()
        if bus[eventName] then
            bus[eventName].listeners[id] = nil
        end
    end
end

--- Fire a named event, passing optional data to all listeners.
function EventSystem.fire(eventName: string, data: any?)
    assert(type(eventName) == "string" and eventName ~= "",
        "[EventSystem] eventName must be a non-empty string")

    local entry = bus[eventName]
    if not entry then return end -- no listeners; safe no-op

    -- Snapshot listeners before iterating to allow mid-fire unsubscription
    local snapshot: {Listener} = {}
    for _, listener in entry.listeners do
        table.insert(snapshot, listener)
    end

    for _, listener in snapshot do
        local ok, err = pcall(listener, data)
        if not ok then
            warn("[EventSystem] Listener error on event:", eventName, "\n", err)
        end
    end
end

--- Fire asynchronously — each listener runs in its own task.spawn.
--- Use when listeners may yield (e.g., TweenService, wait, async IO).
function EventSystem.fireAsync(eventName: string, data: any?)
    assert(type(eventName) == "string" and eventName ~= "",
        "[EventSystem] eventName must be a non-empty string")

    local entry = bus[eventName]
    if not entry then return end

    local snapshot: {Listener} = {}
    for _, listener in entry.listeners do
        table.insert(snapshot, listener)
    end

    for _, listener in snapshot do
        task.spawn(function()
            local ok, err = pcall(listener, data)
            if not ok then
                warn("[EventSystem] Async listener error on event:", eventName, "\n", err)
            end
        end)
    end
end

--- Remove all listeners for a specific event. Use on round/map reset.
function EventSystem.clear(eventName: string)
    bus[eventName] = nil
end

--- Remove all listeners for all events. Full reset.
function EventSystem.clearAll()
    table.clear(bus)
end

--- Returns the number of active listeners for a given event.
function EventSystem.listenerCount(eventName: string): number
    local entry = bus[eventName]
    if not entry then return 0 end
    local count = 0
    for _ in entry.listeners do count += 1 end
    return count
end

--- Debug: print all registered events and listener counts.
function EventSystem.debugDump()
    print("── EventSystem Bus ──")
    for name, entry in bus do
        local count = 0
        for _ in entry.listeners do count += 1 end
        print(string.format("  [%s] → %d listener(s)", name, count))
    end
    print("─────────────────────")
end

return EventSystem
```

---

## 5. BASIC USAGE PATTERNS

### 5.1 Listen and Fire

```lua
local EventSystem    = require(ReplicatedStorage.EventSystem)
local EventRegistry  = require(ReplicatedStorage.EventRegistry)

-- System B: subscribe before firing happens
local unsub = EventSystem.listen(EventRegistry.ENEMY_DIED, function(data)
    print("Enemy died:", data.enemyName, "killed by:", data.killerName)
    -- update score, spawn loot, play sound, etc.
end)

-- System A: fire the event with payload
EventSystem.fire(EventRegistry.ENEMY_DIED, {
    enemyName  = "Goblin",
    killerName = "Player1",
    position   = Vector3.new(10, 0, 5),
})

-- Later — cleanup when system tears down
unsub()
```

---

### 5.2 Async Fire (listeners may yield)

```lua
-- Use fireAsync when listeners run TweenService, task.wait, etc.
EventSystem.fireAsync(EventRegistry.ROUND_ENDED, { winner = "TeamA" })
```

---

### 5.3 CollectionService Integration — Tag + Event Together

```lua
-- Component fires an event on relevant gameplay action
-- ModuleScript: Components/EnemyComponent.lua

local CollectionService = game:GetService("CollectionService")
local EventSystem       = require(ReplicatedStorage.EventSystem)
local EventRegistry     = require(ReplicatedStorage.EventRegistry)

local EnemyComponent = {}
local connections: {[Instance]: {RBXScriptConnection}} = {}

function EnemyComponent.attach(instance: Instance)
    local humanoid = instance:WaitForChild("Humanoid", 5)
    if not humanoid then
        warn("[EnemyComponent] No Humanoid found on", instance:GetFullName())
        return
    end

    local conns = {}

    -- Fire EventSystem when this tagged enemy dies
    table.insert(conns, humanoid.Died:Connect(function()
        EventSystem.fire(EventRegistry.ENEMY_DIED, {
            enemyName = instance.Name,
            position  = instance:GetPivot().Position,
        })
        -- Remove tag so component detaches cleanly
        CollectionService:RemoveTag(instance, "Enemy")
    end))

    connections[instance] = conns
end

function EnemyComponent.detach(instance: Instance)
    local conns = connections[instance]
    if not conns then return end
    for _, c in conns do c:Disconnect() end
    connections[instance] = nil
end

return EnemyComponent
```

---

### 5.4 Round Lifecycle Events

```lua
-- RoundManager fires lifecycle events
local EventSystem   = require(ReplicatedStorage.EventSystem)
local EventRegistry = require(ReplicatedStorage.EventRegistry)

local function startRound(roundNumber: number)
    EventSystem.fire(EventRegistry.ROUND_STARTED, { round = roundNumber })
end

local function endRound(winner: string)
    EventSystem.fire(EventRegistry.ROUND_ENDED, { winner = winner })
    -- Clear all round-scoped event listeners
    EventSystem.clear(EventRegistry.ENEMY_DIED)
    EventSystem.clear(EventRegistry.PLAYER_DAMAGED)
end
```

---

## 6. LIFECYCLE & CLEANUP RULES

Every `EventSystem.listen()` call **must** be paired with an `unsub()` call on teardown.
Failing to unsubscribe causes ghost listeners that fire on stale or destroyed instances.

```lua
-- Pattern: store all unsubs in a table, clear on teardown
local unsubs: {Unsubscribe} = {}

table.insert(unsubs, EventSystem.listen(EventRegistry.ROUND_STARTED, onRoundStart))
table.insert(unsubs, EventSystem.listen(EventRegistry.ENEMY_DIED, onEnemyDied))
table.insert(unsubs, EventSystem.listen(EventRegistry.ITEM_COLLECTED, onItemCollected))

-- Teardown (called on round end, system shutdown, or player leave)
local function cleanup()
    for _, unsub in unsubs do
        unsub()
    end
    table.clear(unsubs)
end
```

---

## 7. PERFORMANCE PRINCIPLES

| Principle | Rule |
|-----------|------|
| **Snapshot before iterate** | Copy listener table before firing to allow safe mid-fire unsubscription |
| **pcall every listener** | One bad listener never blocks others from receiving the event |
| **fireAsync for yielding** | Never yield inside a synchronous `fire()` listener — use `fireAsync` |
| **clear() on scope end** | Remove all listeners for round-scoped events when the round ends |
| **No data stored in bus** | EventSystem only routes; payload data lives in the caller's scope |
| **Avoid high-frequency events** | Do not fire events on `RunService.Heartbeat`; use direct calls or shared state instead |

---

## 8. COMMON ERRORS & SOLUTIONS

### Error 1 — Listener fires after system is destroyed

**Symptom**: Error inside a listener referencing a variable that is `nil` after teardown.

**Cause**: `unsub()` was never called when the owning system cleaned up.

**Fix**: Always call `unsub()` in the system's teardown function. Use the unsub table pattern (Section 6).

---

### Error 2 — One listener error silently blocks others

**Symptom**: Some listeners receive the event; others do not.

**Cause**: A listener throws an error with no `pcall`, halting iteration.

**Fix**: The `EventSystem` implementation wraps every listener in `pcall`. If you wrote a custom
bus without `pcall`, add it. Never assume listener code is error-free.

---

### Error 3 — Listener yields inside synchronous fire()

**Symptom**: `fire()` call hangs; downstream code after `fire()` is delayed.

**Cause**: A listener calls `task.wait()`, `TweenService:Create():Play()`, or any yielding function
inside a synchronous `listen()` callback.

**Fix**: Use `EventSystem.fireAsync()` when any listener might yield.

```lua
-- BAD: yielding inside sync fire
EventSystem.listen("RoundEnded", function(data)
    task.wait(2)             -- blocks fire() caller
    showResultsUI(data)
end)

-- GOOD: use fireAsync
EventSystem.fireAsync("RoundEnded", data)
-- OR: spawn inside listener
EventSystem.listen("RoundEnded", function(data)
    task.spawn(function()
        task.wait(2)
        showResultsUI(data)
    end)
end)
```

---

### Error 4 — Ghost listeners accumulate across round resets

**Symptom**: After 3 rounds, `debugDump()` shows 30+ listeners on the same event. Side effects trigger multiple times per fire.

**Cause**: `listen()` is called on each round start, but `unsub()` is never called on round end.

**Fix**: Call `EventSystem.clear(eventName)` for all round-scoped events on round end,
OR store unsub handles and call them individually.

---

### Error 5 — Firing an event before any listener is registered

**Symptom**: Event fires but nothing reacts.

**Cause**: The firing system initializes and fires before the listening system connects its listener.

**Fix**: Ensure listening systems register in `listen()` before the game logic begins. Use a
loader script that requires all systems in the correct order, or implement a deferred event
(store last fired payload and replay it to new listeners).

```lua
-- Deferred/replay pattern for late subscribers
local lastPayload: {[string]: any} = {}

function EventSystem.firePersistent(eventName: string, data: any?)
    lastPayload[eventName] = data
    EventSystem.fire(eventName, data)
end

function EventSystem.listenWithReplay(eventName: string, listener: Listener): Unsubscribe
    if lastPayload[eventName] ~= nil then
        task.spawn(listener, lastPayload[eventName]) -- replay immediately
    end
    return EventSystem.listen(eventName, listener)
end
```

---

### Error 6 — Cross-boundary event confusion (server vs client)

**Symptom**: Client fires an event expecting server systems to react; nothing happens.

**Cause**: `EventSystem` is in-process only. A client-side `fire()` never reaches the server bus.

**Fix**: For server↔client communication, use `RemoteEvent`. EventSystem is for within-boundary
decoupling only.

```lua
-- WRONG: client fires EventSystem expecting server reaction
EventSystem.fire("PlayerDied", data)  -- server never sees this

-- RIGHT: client fires RemoteEvent; server handles it then fires its own EventSystem
-- Client:
PlayerDiedRemote:FireServer(data)
-- Server:
PlayerDiedRemote.OnServerEvent:Connect(function(player, data)
    EventSystem.fire(EventRegistry.PLAYER_DIED, { player = player, data = data })
end)
```

---

## 9. DEBUG & VALIDATION UTILITIES

```lua
-- Dump entire event bus state
EventSystem.debugDump()
-- Output:
-- ── EventSystem Bus ──
--   [EnemyDied]     → 3 listener(s)
--   [RoundStarted]  → 2 listener(s)
--   [ItemCollected] → 1 listener(s)
-- ─────────────────────

-- Check listener count for a specific event
local count = EventSystem.listenerCount(EventRegistry.ENEMY_DIED)
print("EnemyDied listeners:", count)

-- Assert at least one listener exists before firing (dev guard)
local function assertHasListeners(eventName: string)
    assert(EventSystem.listenerCount(eventName) > 0,
        "[EventSystem] No listeners registered for: " .. eventName)
end
```

---

## 10. QUICK REFERENCE

```
EventSystem.listen(eventName, listener)        → subscribe; returns unsub()
EventSystem.fire(eventName, data?)             → fire synchronously to all listeners
EventSystem.fireAsync(eventName, data?)        → fire; each listener in task.spawn
EventSystem.clear(eventName)                   → remove all listeners for one event
EventSystem.clearAll()                         → remove all listeners for all events
EventSystem.listenerCount(eventName)           → number of active listeners
EventSystem.debugDump()                        → print full bus to output
```

**Golden rules**:
- Always store the `unsub` handle returned by `listen()` and call it on teardown.
- Use `EventRegistry` constants — never inline event name strings.
- Use `fireAsync` when any listener may yield.
- Call `clear()` on round-scoped events when the round ends.
- EventSystem is in-process only — cross-boundary needs `RemoteEvent`.
- Never store game state inside the EventSystem bus itself.