# Luau Patterns

## Module Shape

```lua
local Module = {}

function Module:Init()
end

function Module:Start()
end

function Module:Destroy()
end

return Module
```

## Typed API

```lua
export type Result = {
    ok: boolean,
    message: string?,
}

local function makeResult(ok: boolean, message: string?): Result
    return {
        ok = ok,
        message = message,
    }
end
```

## Connection Bag

```lua
local ConnectionBag = {}
ConnectionBag.__index = ConnectionBag

function ConnectionBag.new()
    return setmetatable({
        _items = {},
    }, ConnectionBag)
end

function ConnectionBag:Add(connection: RBXScriptConnection)
    table.insert(self._items, connection)
    return connection
end

function ConnectionBag:Destroy()
    -- PUTUS KONEKSI
    for _, connection in self._items do
        connection:Disconnect()
    end

    table.clear(self._items)
end

return ConnectionBag
```

## State Machine

```lua
local StateMachine = {}
StateMachine.__index = StateMachine

function StateMachine.new(initial: string, transitions: { [string]: { [string]: string } })
    return setmetatable({
        _state = initial,
        _transitions = transitions,
    }, StateMachine)
end

function StateMachine:Get(): string
    return self._state
end

function StateMachine:Send(event: string): boolean
    -- CEK TRANSISI
    local group = self._transitions[self._state]
    if not group then
        return false
    end

    local nextState = group[event]
    if not nextState then
        return false
    end

    self._state = nextState
    return true
end

return StateMachine
```

## Require Rule

- Require by Roblox instance name.
- Do not include file extension.
- Do not require `init`.
- Use `WaitForChild` across Roblox service boundaries.
- Use direct child access only for guaranteed local descendants.
