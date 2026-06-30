# Performance

## Purpose

- This file covers Basic performance, networking, and gameplay flow.
- Read this file for `performance`, `networking`, `gameplay`, remotes, lag, FPS, loops, cleanup, gameplay authority, combat, shop, carry, donation, and similar lightweight systems.

## Always

- Cache services at top.
- Cache repeated instance paths.
- Use events before loops.
- Use `task.wait` only when polling is unavoidable.
- Batch remote updates.
- Batch DataStore writes.
- Reuse UI and effects.
- Destroy temporary instances.
- Disconnect connections.
- Clear tables on cleanup.

## Never

- Do not fire remotes every frame.
- Do not create instances every frame.
- Do not use `RenderStepped` on server.
- Do not call `FindFirstChild` in hot loops.
- Do not save DataStore per action.
- Do not leave player tables after `PlayerRemoving`.

## Cleanup Template

```lua
local connections: { RBXScriptConnection } = {}

local function track(connection: RBXScriptConnection)
    table.insert(connections, connection)
    return connection
end

local function cleanup()
    -- CLEAN CONNECTIONS
    for _, connection in connections do
        connection:Disconnect()
    end

    table.clear(connections)
end
```

## Networking

### Remote Names

- `System_Action_RE` for `RemoteEvent`.
- `System_Action_RF` for `RemoteFunction`.
- `System_Action_BE` for `BindableEvent`.
- Keep remotes under `ReplicatedStorage/SystemName/Remotes`.
- Prefer one typed action remote over many small remotes when safe.

### Remote Tree

```text
ReplicatedStorage/
  ShopSystem/
    Remotes/
      Shop_Action_RE
      Shop_GetInfo_RF
      Shop_Update_BE
```

### Remote Creation

```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local systemFolder = ReplicatedStorage:FindFirstChild("ShopSystem") or Instance.new("Folder")
systemFolder.Name = "ShopSystem"
systemFolder.Parent = ReplicatedStorage

local remotesFolder = systemFolder:FindFirstChild("Remotes") or Instance.new("Folder")
remotesFolder.Name = "Remotes"
remotesFolder.Parent = systemFolder

local function ensureRemote(className: string, name: string): Instance
    local remote = remotesFolder:FindFirstChild(name)
    if remote and remote.ClassName == className then
        return remote
    end

    if remote then
        remote:Destroy()
    end

    remote = Instance.new(className)
    remote.Name = name
    remote.Parent = remotesFolder
    return remote
end

local Shop_Action_RE = ensureRemote("RemoteEvent", "Shop_Action_RE") :: RemoteEvent
local Shop_GetInfo_RF = ensureRemote("RemoteFunction", "Shop_GetInfo_RF") :: RemoteFunction
```

### Server Remote Rule

- Validate player identity.
- Validate argument types.
- Validate ownership.
- Validate distance when spatial.
- Validate cooldown.
- Never accept currency, inventory, damage, or rank from client as truth.

### Typed Action Pattern

```lua
type ActionPayload = {
    action: string,
    data: { [string]: any }?,
}

local handlers: { [string]: (Player, { [string]: any }) -> () } = {}

Shop_Action_RE.OnServerEvent:Connect(function(player: Player, payload: ActionPayload)
    -- VALIDATE PAYLOAD
    if type(payload) ~= "table" then
        return
    end

    -- VALIDATE ACTION
    if type(payload.action) ~= "string" then
        return
    end

    local handler = handlers[payload.action]
    if not handler then
        return
    end

    handler(player, payload.data or {})
end)
```

### Rate Limit

```lua
local calls: { [Player]: number } = {}
local last: { [Player]: number } = {}

local function blocked(player: Player, limit: number, window: number): boolean
    local now = os.clock()
    if not last[player] or now - last[player] > window then
        last[player] = now
        calls[player] = 1
        return false
    end

    calls[player] += 1
    return calls[player] > limit
end
```

## Gameplay

### Rules

- Server owns damage, rewards, inventory, rank, currency, and cooldown.
- Client owns camera, input, local animation, visual feedback, and UI.
- Validate distance for touch, pickup, trade, combat, carry, and interact systems.
- Use attributes for simple replicated state.
- Use modules for reusable gameplay logic.
- Use configs for tunable values.

### Common Systems

- Carry: server validates target, distance, cooldown, alive state, and release.
- Dance: client plays animation; server may replicate emote state.
- Shop: server validates item, price, stock, ownership, and currency.
- Donation: server validates product receipt and grants reward once.
- Relationship: server validates request, consent, cooldown, and persistence.
- Combat: server validates weapon, range, state, team, cooldown, and damage.

### State Names

- Use short explicit states: `Idle`, `Busy`, `Trading`, `Carrying`, `Stunned`, `Dead`.
- Do not store gameplay truth only in UI.
- Clear state on death, leaving, teleport, or system shutdown.
