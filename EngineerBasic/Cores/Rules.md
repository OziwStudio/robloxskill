# Core Rules

## General

- Write production-ready Luau.
- Never trust the client.
- Validate all remote input on the server.
- Track and clean every connection.
- Use `task.*`; never use `wait`, `spawn`, or `delay`.
- Use typed public APIs.
- Avoid placeholders in `!genfull`.
- Avoid `TODO` and `FIXME` tags for now.
- If whitelist, admin, or permission data is needed, ask the user to provide the `UserId`.
- Default generate priority: working code first, low performance cost second, complex architecture later.
- Do not overdesign a system unless the user asks for deeper complexity.
- When making a system, prefer simple, correct, and light output before advanced optimization.

## Complexity Classifier

- Classify every requested system before coding: `Simple`, `Moderate`, or `Complex`.
- `Simple`: one direct script, no shared state, no DataStore, zero or one remote.
- `Moderate`: server-client flow, config, remotes, modules, or two to five files.
- `Complex`: DataStore, inventory, combat, economy, admin, matchmaking, anti-cheat, many modules, or cross-system state.
- `Simple` may use root directory when clearer.
- `Moderate` and `Complex` must use system folders.
- State the class before the file tree.

## System Folder Rules

- Use `ReplicatedStorage/SystemName` for shared assets.
- Use `ServerScriptService/SystemServer` for server files.
- Use `StarterPlayerScripts/SystemClient` for client files.
- Use `StarterGui/SystemGui` for GUI instances when needed.
- Use `StarterCharacterScripts/SystemCharacter` for character scripts when needed.
- Do not place moderate or complex system files directly in service roots.
- Simple systems may use root files, example: `ServerScriptService/PlayerCollision.server.luau`.

## ReplicatedStorage Layout

- `ReplicatedStorage/SystemName/Remotes` stores all remotes.
- `ReplicatedStorage/SystemName/Modules` stores shared modules.
- `ReplicatedStorage/SystemName/ConfigSystemName.luau` is used for one shared config.
- `ReplicatedStorage/SystemName/ConfigShared` is used for multiple shared config files.

## Server Layout

- `ServerScriptService/SystemServer/Main.server.luau` is the default entry script.
- `ServerScriptService/SystemServer/Main.legacy.luau` is allowed for ordinary direct-run script compatibility.
- `ServerScriptService/SystemServer/ConfigSystem.luau` is allowed for server-safe config.
- Use `Services`, `Cores`, or `Handlers` only when there are multiple module files.
- Put a single server module directly under the system folder.

## Client Layout

- `StarterPlayerScripts/SystemClient/Main.local.luau` is the default local entry script.
- `StarterPlayerScripts/SystemClient/Main.client.luau` is secondary compatibility.
- Keep input, UI, camera, and effects separated only when needed.

## File Types

- `.luau` is the main extension.
- `.lua` is secondary.
- `.server.luau` means Server Script.
- `.legacy.luau` means ordinary Script compatibility.
- `.local.luau` means LocalScript.
- `.client.luau` means LocalScript compatibility.
- `.luau` without type suffix means ModuleScript.

## Folder Init Rule

- Use folder/init when a script must have children outside Roblox Studio.
- Example outside Studio:

```text
MainScript/
  init.server.luau
  CoreScript.luau
```

- Treat `MainScript/init.server.luau` as `MainScript.server.luau`.
- Do not require file extensions.
- Do not require `init`.
- Correct: `require(parent:WaitForChild("MainScript"):WaitForChild("CoreScript"))`.
- Wrong: `require(parent:WaitForChild("MainScript.server.luau"))`.
- Wrong: `require(parent:WaitForChild("init"))`.

## Header Rule

- Every script file starts with this header:

```lua
--[[
# SystemName v1.0.0
# Path/File.server.luau (server script)(v1.0.0)
# Side: Server
# Deps: ReplicatedStorage.SystemName.Modules.ModuleName
-----
Fix: -
Updt: -
Add: -
]]
```

- Use `Fix`, `Updt`, and `Add` only when relevant.
- Use one line when short: `Fix: ... / Updt: ... / Add: ...`.

## Footer Rule

- Every script file ends with this footer:

```lua
--[[
# SystemName v1.0.0
# Path/File.server.luau (server script)(v1.0.0)
]]
```

## Version Rule

- Start new generated systems at `v1.0.0`.
- Increase patch for every generated edit: `v1.0.0` to `v1.0.1`.
- Increase minor when user confirms a batch as final: `v1.1.0` to `v1.2.0`.
- Increase major for breaking change or large refactor: `v1.x.x` to `v2.0.0`.
- Update header and footer versions together.

## Comment Rule

- Add short comments to each meaningful logic block.
- Comment length must be one to five words.
- Important logic comments must be uppercase.
- If the user prompts in Indonesian, comments use Indonesian.
- If the user prompts in another language, comments use English.
- Do not over-comment obvious single lines.

## Debug Rule

- Use `local DebugSucces = true` or `false` for success logs.
- Error logs are always active and hardcoded.
- Success logs are gated by `DebugSucces`.
- Debug prefix format: `[SystemName][Division]Err:message`.
- Success prefix format: `[SystemName][Division]OK:message`.
- Messages must be short.

## Remote Rule

- Put remotes in `ReplicatedStorage/SystemName/Remotes`.
- Use `System_Action_RE` for `RemoteEvent`.
- Use `System_Action_RF` for `RemoteFunction`.
- Use `System_Action_BE` for `BindableEvent`.
- Example: `Shop_Status_RE`.
- Example: `Shop_GetInfo_RF`.
- Example: `Shop_Update_BE`.
- Use fewer remotes when one typed remote can safely route actions.

## Output Rule

- `!genfull` always starts with classification and file tree before code.
- Show complete paths before script blocks.
- Do not generate code when the user explicitly says to wait.
- When the user asks to create a system, ask these questions first:
  - Do you already have a reference system/code? Attach it if available.
  - Are you using a framework, project management, or similar support?
  - What complexity depth do you want: Shallow, Medium, or Deep?
  - Generation depth: separated or all-in.
  - Generation type: package or copy-paste.
