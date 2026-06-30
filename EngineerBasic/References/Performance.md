# Performance

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
    -- BERSIHKAN KONEKSI
    for _, connection in connections do
        connection:Disconnect()
    end

    table.clear(connections)
end
```
