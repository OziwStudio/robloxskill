# Systems

## System Blueprint Rule

- Start from classifier.
- Build file tree first.
- Put shared modules in `ReplicatedStorage/SystemName`.
- Put server authority in `ServerScriptService/SystemServer`.
- Put client input and visuals in `StarterPlayerScripts/SystemClient`.
- Use config for tunable values.
- Use remotes only for client-server boundary.

## Shop System

```text
ReplicatedStorage/
  ShopSystem/
    Remotes/
      Shop_Action_RE
      Shop_GetInfo_RF
    Modules/
      ShopTypes.luau
    ConfigShop.luau

ServerScriptService/
  ShopServer/
    Main.server.luau
    ShopService.luau

StarterPlayerScripts/
  ShopClient/
    Main.local.luau
    ShopController.luau
```

Server validates item, price, currency, stock, ownership, cooldown, and receipt.

## Carry System

```text
ReplicatedStorage/
  CarrySystem/
    Remotes/
      Carry_Action_RE
    ConfigCarry.luau

ServerScriptService/
  CarryServer/
    Main.server.luau
    CarryService.luau

StarterPlayerScripts/
  CarryClient/
    Main.local.luau
    CarryController.luau
```

Server validates distance, alive state, consent, cooldown, target state, and release.

## Dance System

```text
ReplicatedStorage/
  DanceSystem/
    Remotes/
      Dance_Action_RE
    ConfigDance.luau

ServerScriptService/
  DanceServer/
    Main.server.luau
    DanceService.luau

StarterPlayerScripts/
  DanceClient/
    Main.local.luau
    DanceController.luau
```

Client handles local animation. Server validates emote id, cooldown, and replicated state.

## Donation System

```text
ReplicatedStorage/
  DonationSystem/
    Remotes/
      Donation_Status_RE
    ConfigDonation.luau

ServerScriptService/
  DonationServer/
    Main.server.luau
    ReceiptHandler.luau
    ConfigDonation.luau
```

Server grants reward only from valid Marketplace receipt.

## Relationship System

```text
ReplicatedStorage/
  RelationshipSystem/
    Remotes/
      Relationship_Action_RE
      Relationship_GetInfo_RF
    Modules/
      RelationshipTypes.luau

ServerScriptService/
  RelationshipServer/
    Main.server.luau
    Services/
      RelationshipService.luau
      RelationshipData.luau

StarterPlayerScripts/
  RelationshipClient/
    Main.local.luau
    RelationshipController.luau
```

Server validates consent, cooldown, target, state, and persistence.
