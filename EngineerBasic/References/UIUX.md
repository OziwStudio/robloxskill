# UIUX

## UI Rules

- Client creates and controls UI.
- Server never trusts UI state.
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
    -- BUAT ROOT
    local gui = Instance.new("ScreenGui")
    gui.Name = "ShopSystem_v1"
    gui.ResetOnSpawn = false
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.Parent = playerGui
    return gui
end

local function destroyRoot(gui: ScreenGui?)
    -- HAPUS UI
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
