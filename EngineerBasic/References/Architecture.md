# Architecture

## Classifier Output

Use this before code:

```text
Class: Moderate
Reason: server-client flow, remotes, shared config.
```

## Simple Layout

Use only when the system is direct and isolated.

```text
ServerScriptService/
  Example.server.luau
```

## Moderate Layout

```text
ReplicatedStorage/
  ExampleSystem/
    Remotes/
      Example_Action_RE
      Example_GetInfo_RF
    Modules/
      ExampleTypes.luau
    ConfigExample.luau

ServerScriptService/
  ExampleServer/
    Main.server.luau
    ConfigExample.luau
    ExampleService.luau

StarterPlayerScripts/
  ExampleClient/
    Main.local.luau
    ExampleController.luau
```

## Complex Layout

```text
ReplicatedStorage/
  GenericSystem/
    Remotes/
    Modules/
    ConfigShared/

ServerScriptService/
  GenericServer/
    Main.server.luau
    Services/
    Cores/
    Handlers/
    ConfigGeneric.luau

StarterPlayerScripts/
  GenericClient/
    Main.local.luau
    Controllers/
    UI/
```

## Server Entry Template

```lua
--[[
# SystemName v1.0.0
# ServerScriptService/SystemServer/Main.server.luau (server script)(v1.0.0)
# Side: Server
# Deps: ServerScriptService.SystemServer.SystemService
-----
Add: server bootstrap
]]

local DebugSucces = true

local SystemService = require(script.Parent:WaitForChild("SystemService"))

local function logOK(message: string)
    if DebugSucces then
        print("[SystemName][Server]OK:" .. message)
    end
end

local function logErr(message: string)
    warn("[SystemName][Server]Err:" .. message)
end

-- MULAI SERVER
local ok, err = pcall(function()
    SystemService:Init()
    SystemService:Start()
end)

if ok then
    logOK("ready")
else
    logErr(tostring(err))
end

--[[
# SystemName v1.0.0
# ServerScriptService/SystemServer/Main.server.luau (server script)(v1.0.0)
]]
```

## Lifecycle Rule

- `Init()` prepares config, cache, and remotes.
- `Start()` connects events and begins runtime work.
- `Destroy()` disconnects and clears state.
- Do not require sibling services at module top when circular dependency is possible.
- Prefer dependency injection for complex systems.

## Output Map Rule

For `!genfull`, output:

```text
Class: Moderate
File Tree:
game/
  ReplicatedStorage/
    ExampleSystem/
...
```
