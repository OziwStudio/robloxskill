# UIUX

## UI Rules

- Before generating UI, ask:
  - `Do you already have a UI, or should I generate it by script?`
  - `Should the UI generator script be merged with the logic, or separated from logic/UX?`
- If user already has UI, integrate with existing hierarchy.
- If user wants generated UI, create script-generated UI.
- If user wants separate UI generator, keep UI builder separate from logic/controller.
- If user wants merged logic, keep UI section clearly separated inside the same script.
- Client creates and controls UI.
- Server never trusts UI state.
- Name script-bound UI specifically, not `TextLabel`, `Frame`, or `ImageLabel`.
- Use names like `TLbl_PlayerName`, `FramePanel`, `IBtnIcon`, or `ListItems`.
- Use scale-based sizing for mobile.
- Use small reusable builders.
- Clean UI on destroy.
- Disconnect input events.
- Keep decoration minimal unless requested.

## Layout

```text
StarterPlayerScripts/
  ShopClient/
    Main.local.luau
    UI/
      ShopUI.luau
    Controllers/
      ShopController.luau

ReplicatedStorage/
  ShopSystem/
    ConfigShop.luau
```

## Client UI Template

```lua
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local function createRoot(): ScreenGui
    -- CREATE ROOT
    local gui = Instance.new("ScreenGui")
    gui.Name = "ShopSystem_v1"
    gui.ResetOnSpawn = false
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.Parent = playerGui
    return gui
end

local function destroyRoot(gui: ScreenGui?)
    -- DESTROY UI
    if gui then
        gui:Destroy()
    end
end
```

## Input Rule

- Use `UserInputService` for hotkeys.
- Use `ContextActionService` for gameplay actions.
- Do not bind duplicate actions.
- Unbind on destroy.

## Remote UI Rule

- UI sends intent only.
- UI never sends final price.
- UI never sends final reward.
- UI never sends trusted rank.
- UI handles loading, success, and error states.
