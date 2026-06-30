# Data

## Data Rules

- Server owns all persistent data.
- Use one profile key per player.
- Use schema version.
- Reconcile missing fields.
- Migrate old schema.
- Save with retry.
- Save on interval.
- Save on `PlayerRemoving`.
- Save on `BindToClose`.
- Mark dirty before save.
- Clear cache after player leaves.

## Layout

```text
ServerScriptService/
  ProfileServer/
    Main.server.luau
    ProfileService.luau
    ConfigProfile.luau

ReplicatedStorage/
  ProfileSystem/
    Modules/
      ProfileTypes.luau
```

## Schema Template

```lua
export type PlayerData = {
    _version: number,
    Coins: number,
    Level: number,
    Inventory: { [string]: number },
}

local DEFAULT_DATA: PlayerData = {
    _version = 1,
    Coins = 0,
    Level = 1,
    Inventory = {},
}
```

## Retry Rule

```lua
local function retry<T>(attempts: number, job: () -> T): (boolean, T?)
    for attempt = 1, attempts do
        local ok, result = pcall(job)
        if ok then
            return true, result
        end

        task.wait(attempt)
    end

    return false, nil
end
```

## Reconcile Rule

```lua
local function cloneTable(value: { [any]: any }): { [any]: any }
    local copy = {}
    for key, item in value do
        copy[key] = type(item) == "table" and cloneTable(item) or item
    end
    return copy
end

local function reconcile(data: { [any]: any }, defaults: { [any]: any })
    for key, default in defaults do
        if data[key] == nil then
            data[key] = type(default) == "table" and cloneTable(default) or default
        elseif type(default) == "table" and type(data[key]) == "table" then
            reconcile(data[key], default)
        end
    end
end
```

## Migration Rule

```lua
local function migrate(data: PlayerData): PlayerData
    -- MIGRASI DATA
    if data._version == 1 then
        data._version = 2
    end

    return data
end
```
