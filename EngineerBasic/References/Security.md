# Security

## Server Authority

- Client may request.
- Server must decide.
- Server validates all inputs.
- Server owns rewards.
- Server owns damage.
- Server owns inventory.
- Server owns currency.
- Server owns permission.

## Required Checks

- Player exists.
- Player is alive when needed.
- Type is valid.
- Value is in range.
- Distance is valid.
- Cooldown is valid.
- Ownership is valid.
- State is valid.
- Permission is valid.

## Validation Helpers

```lua
local Players = game:GetService("Players")

local Validate = {}

function Validate.player(player: any): boolean
    return typeof(player) == "Instance"
        and player:IsA("Player")
        and Players:GetPlayerByUserId(player.UserId) ~= nil
end

function Validate.string(value: any, maxLength: number): boolean
    return type(value) == "string" and #value > 0 and #value <= maxLength
end

function Validate.number(value: any, min: number, max: number): boolean
    return type(value) == "number" and value == value and value >= min and value <= max
end

function Validate.distance(a: BasePart, b: BasePart, maxDistance: number): boolean
    return (a.Position - b.Position).Magnitude <= maxDistance
end

return Validate
```

## Anti-Exploit Rule

- Do not kick from one weak signal.
- Count repeated violations.
- Log short error messages.
- Reset impossible states.
- Kick only confirmed repeat abuse.

## Permission Rule

```lua
local ADMINS = {
    [userId] = true,
}

local function isAdmin(player: Player, userId: number): boolean
    return ADMINS[userId] == true and player.UserId == userId
end
```

## Remote Abuse Rule

```lua
local calls: { [Player]: { count: number, time: number } } = {}

local function isLimited(player: Player, limit: number, window: number): boolean
    local now = os.clock()
    local data = calls[player]

    if not data or now - data.time > window then
        calls[player] = { count = 1, time = now }
        return false
    end

    data.count += 1
    return data.count > limit
end
```
