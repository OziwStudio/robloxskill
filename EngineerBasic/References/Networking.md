# Networking

## Remote Names

- `System_Action_RE` for `RemoteEvent`.
- `System_Action_RF` for `RemoteFunction`.
- `System_Action_BE` for `BindableEvent`.
- Keep remotes under `ReplicatedStorage/SystemName/Remotes`.
- Prefer one typed action remote over many small remotes when safe.

## Remote Tree

```text
ReplicatedStorage/
  ShopSystem/
    Remotes/
      Shop_Action_RE
      Shop_GetInfo_RF
      Shop_Update_BE
```

## Remote Creation

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

## Server Remote Rule

- Validate player identity.
- Validate argument types.
- Validate ownership.
- Validate distance when spatial.
- Validate cooldown.
- Never accept currency, inventory, damage, or rank from client as truth.

## Typed Action Pattern

```lua
type ActionPayload = {
    action: string,
    data: { [string]: any }?,
}

local handlers: { [string]: (Player, { [string]: any }) -> () } = {}

Shop_Action_RE.OnServerEvent:Connect(function(player: Player, payload: ActionPayload)
    -- VALIDASI PAYLOAD
    if type(payload) ~= "table" then
        return
    end

    -- VALIDASI AKSI
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

## Rate Limit

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
