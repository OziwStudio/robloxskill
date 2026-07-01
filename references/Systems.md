# Systems

## Purpose
- Use for common Roblox system blueprints and complete-system planning.
- Use `references/bridges/ImplementationPatterns.md` when the blueprint needs to become script directly.

## System Blueprint
1. Define owner.
2. Define state.
3. Define remotes.
4. Define config.
5. Define data schema.
6. Define UI flow.
7. Define security gates.
8. Define test plan.

## System Contract
```text
System:
Owner:
Complexity:
Server State:
Persistent Data:
Client State:
Remotes:
Configs:
Security Gates:
Performance Gates:
Tests:
```

## Common Systems
- DataManager.
- RemoteHandler.
- CurrencyService.
- InventoryService.
- ShopService.
- CombatService.
- QuestService.
- RoundManager.
- LobbyManager.
- UIController.
- VFXController.

## Build Order
1. Shared types and constants.
2. Remotes.
3. Server state and services.
4. Client controllers.
5. UI.
6. Tests.
7. Publish gates.

## Service Pattern
- Server service owns state mutation.
- Shared modules own types, constants, and pure helpers.
- Client controller owns input, camera, local display, and UI intent.
- Remote handler validates boundary and calls server service.
- Data service persists only server-approved values.

## Example Remote Surface
```text
Remotes/
  System_Action_RE      Client -> Server intent
  System_Sync_RE        Server -> Client state snapshot
  System_GetInfo_RF     Client -> Server read-only query
```

## Example Folder Decision
- Simple: one server script when isolated.
- Moderate: `ReplicatedStorage/SystemName`, `ServerScriptService/SystemServer`, `StarterPlayerScripts/SystemClient`.
- Complex: add `Services`, `Controllers`, `Handlers`, `Modules`, and `ConfigShared`.

## Template Integration
- Use `TemplateBridge.md` for copied game templates.
- Adapt template paths to Roblox Skill Pluz architecture rules.
- Keep headers, footers, comments, debug, and version rules.

## Concrete Patterns
- Module service: `references/bridges/ImplementationPatterns.md#Module Shape`.
- Remote map: `references/bridges/ImplementationPatterns.md#Remote Map`.
- Remote handler: `references/bridges/ImplementationPatterns.md#Remote Handler`.
- Data schema: `references/bridges/ImplementationPatterns.md#Profile Schema`.
