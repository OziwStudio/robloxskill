# TeleportService
> Roblox Luau reference for player transport, place transitions, reserved servers, and teleport safety.
> Use this file as the single TeleportService guide for AI reasoning.

## 1. Purpose

TeleportService moves players between places or into reserved servers.
It is used for matchmaking, lobbies, party travel, and multi-place experiences.

Use it when you want:
- place-to-place travel
- reserved server handoff
- teleport data transfer
- transition UI control

Do not use it for:
- same-server movement
- client-authority teleport logic
- persistent game state

## 2. Core Principles

1. Teleport requests are server-owned.
2. Teleport flow is asynchronous.
3. Teleport data is transient.
4. Critical state belongs in a datastore, not teleport payloads.
5. The destination server must validate received data.
6. Reserved servers help keep parties or matches isolated.
7. Teleport failures must be handled.

## 3. API Surface

### 3.1 TeleportAsync

`TeleportService:TeleportAsync(placeId, players, teleportOptions)`

- modern teleport API
- supports options and teleport data
- should be wrapped in error handling

### 3.2 GetLocalPlayerTeleportData

`TeleportService:GetLocalPlayerTeleportData()`

- reads incoming teleport data on the destination server
- should be handled early in server startup

### 3.3 SetTeleportGui

`TeleportService:SetTeleportGui(guiObject)`

- sets the transition UI
- should be called before teleport starts

### 3.4 TeleportFailed

`TeleportService.TeleportFailed`

- fires when teleport fails
- should always be logged and handled

## 4. Flow Rules

| Action | Server | Client |
|---|---|---|
| teleport request | authoritative | blocked |
| teleport data | write | read-only on arrival |
| transition UI | configured by server | displayed locally |

Rules:
- the server decides who teleports
- the client only receives the transition experience
- destination data is one-time context, not permanent state

## 5. Teleport Pattern

The correct flow is:
1. validate the request
2. prepare transition UI
3. prepare teleport data
4. call `TeleportAsync`
5. handle failure
6. restore the user flow if needed

```lua
local TeleportService = game:GetService("TeleportService")

local function safeTeleport(players: {Player}, placeId: number, data: any)
    local options = Instance.new("TeleportOptions")
    options:SetTeleportData(data)

    local ok, err = pcall(function()
        TeleportService:TeleportAsync(placeId, players, options)
    end)

    if not ok then
        warn("Teleport failed:", err)
    end
end
```

## 6. Teleport Data Rules

Teleport data is for short-lived context only.

Rules:
- keep payloads small
- use serializable data only
- do not store critical inventory or economy state
- treat all payloads as untrusted on arrival
- use datastore-backed state for anything permanent

Good uses:
- selected map
- party join context
- destination mode

Bad uses:
- gold balance
- inventory ownership
- permanent unlocks

## 7. Reserved Server Rules

Reserved servers are useful for private matches and party sessions.

Rules:
- reserve only when you need isolation
- cache access codes when appropriate
- clear match state when the session ends
- do not call reserve logic in high-frequency loops

## 8. UX Rules

Teleporting should feel deliberate.

Rules:
- show a transition state
- use branded loading UI
- inform the player that a handoff is happening
- avoid abrupt blank-screen transitions

## 9. Security Rules

- never expose teleport authority to the client
- never trust remote-triggered teleport requests without validation
- verify lobby state, party membership, or permission before teleporting
- validate received teleport data on the destination server

## 10. Performance Rules

- avoid teleport spam
- batch when needed
- do not reserve servers in hot loops
- expect throttling under heavy use

## 11. Common Failures

### 11.1 Lost state

Problem: teleport data is used for essential inventory.
Fix: load real state from datastore.

### 11.2 Infinite loading

Problem: failure is not handled.
Fix: always handle `TeleportFailed`.

### 11.3 Shutdown race

Problem: server closes too early.
Fix: clean up or wait inside shutdown flow.

### 11.4 Unauthorized teleport

Problem: client can request any place.
Fix: validate server-side before teleport.

## 12. Professional Behavior Rules for AI

- prioritize reliability over brevity
- keep source and destination logic separate
- warn against teleport data as storage
- keep teleport options idiomatic
- remind users that teleport is network-heavy

## 13. Minimal Checklist

- server validates the request
- `pcall` wraps `TeleportAsync`
- `TeleportFailed` is handled
- custom teleport UI is set
- teleport data is minimal
- reserved server codes are managed
- destination validates received data

## 14. Final Rule

TeleportService is a bridge, not a storage container.
Use it to move players cleanly, and keep the real state in the right place.

