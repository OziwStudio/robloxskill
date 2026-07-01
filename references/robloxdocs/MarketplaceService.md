# MarketplaceService
> Roblox Luau reference for monetization, purchase flow, receipt handling, and server-authoritative commerce.
> Use this file as the single MarketplaceService guide for AI reasoning.

## 1. Purpose

MarketplaceService handles digital purchases inside a Roblox experience.
It is used for developer products, game passes, product metadata, and transaction callbacks.

Use it when you want:
- consumable purchases
- permanent unlocks
- product metadata lookup
- safe receipt processing

Do not use it for:
- local game state
- client-driven purchase logic
- non-purchase communication

## 2. Core Principles

1. Purchase fulfillment belongs to the server.
2. `ProcessReceipt` is the source of truth for developer products.
3. The client may prompt a purchase, but never decide the outcome.
4. Every successful transaction must be persisted safely.
5. Receipt replay is possible, so idempotency matters.
6. Product metadata should be cached when possible.
7. Game pass ownership should be checked on the server.

## 3. API Surface

### 3.1 PromptPurchase

`MarketplaceService:PromptPurchase(player, assetId)`

- shows the purchase UI
- use for prompted purchase flow only

### 3.2 PromptGamePassPurchase

`MarketplaceService:PromptGamePassPurchase(player, gamePassId)`

- shows the game pass purchase UI
- use only after server-side intent checks

### 3.3 GetProductInfo

`MarketplaceService:GetProductInfo(assetId, infoType)`

- fetches product metadata
- cache results to avoid repeated calls

### 3.4 ProcessReceipt

`MarketplaceService.ProcessReceipt`

- server callback for developer products
- must always return a valid purchase decision

## 4. ProcessReceipt Pattern

This callback is the heart of monetization.
If it fails to return a decision, Roblox can retry the receipt later.

```lua
local MarketplaceService = game:GetService("MarketplaceService")
local Players = game:GetService("Players")

MarketplaceService.ProcessReceipt = function(receiptInfo)
    local player = Players:GetPlayerByUserId(receiptInfo.PlayerId)
    if not player then
        return Enum.ProductPurchaseDecision.NotProcessedYet
    end

    local success = grantItemToPlayer(player, receiptInfo.ProductId)
    if success then
        return Enum.ProductPurchaseDecision.PurchaseGranted
    end

    return Enum.ProductPurchaseDecision.NotProcessedYet
end
```

## 5. Security Rules

- never trust the client to report purchase success
- store receipt IDs to prevent double grants
- use server-side validation for every purchase outcome
- keep inventory and currency updates server-owned
- use `UpdateAsync` or another safe persistence path for granted rewards
- ignore client-fired purchase claims

## 6. Developer Product vs Game Pass

| Feature | Developer Product | Game Pass |
|---|---|---|
| Type | consumable | permanent |
| Ownership | can be bought multiple times | owned once |
| Logic | `ProcessReceipt` | server ownership check |
| Common use | currency, boosts, items | VIP, access, permanent perks |

Rules:
- use developer products for repeat purchases
- use game passes for permanent unlocks
- do not use developer products for permanent unlocks

## 7. Product Registry

Use a registry table instead of long `if` chains.

```lua
local ProductHandlers = {
    [12345] = function(player)
        grantCurrency(player, 100)
    end,
    [67890] = function(player)
        grantItem(player, "HealthPotion")
    end,
}
```

Rules:
- keep product IDs centralized
- keep handler logic small
- keep receipt flow readable

## 8. Failure Handling

### Player leaves mid-processing

Return `NotProcessedYet` and let Roblox retry later.

### Service outage or error

Return `NotProcessedYet` when the grant cannot be completed safely.

### Duplicate receipt

Check transaction IDs before granting again.

### Missing product mapping

Log the receipt and refuse to grant unknown products.

## 9. UI Rules

- do not spam purchase prompts
- do not show purchase prompts in loops
- check ownership before prompting a game pass
- keep purchase UI separate from reward logic

## 10. Performance Rules

- cache product metadata
- avoid repeated `GetProductInfo` calls
- keep receipt handler short
- do not block the main gameplay loop with purchase work

## 11. Common Failures

### 11.1 Ignoring receipt ID

Problem: same receipt grants twice.
Fix: store and check receipt IDs.

### 11.2 Client-side purchase logic

Problem: the client claims a purchase happened.
Fix: rely only on `ProcessReceipt`.

### 11.3 Missing return

Problem: receipt callback does not return a decision.
Fix: always return a valid enum.

### 11.4 Hardcoded product IDs

Problem: IDs are repeated in many scripts.
Fix: use one constant registry.

## 12. Professional Behavior Rules for AI

- prioritize transaction integrity
- default to server authority
- separate purchase prompt from grant logic
- warn against client trust
- keep receipt flow deterministic
- add comments for safety-critical grant paths

## 13. Minimal Checklist

- `ProcessReceipt` exists on the server
- return values are explicit
- receipt IDs are tracked
- product IDs are centralized
- game pass checks are server-side
- client does not trigger grant logic
- product info is cached

## 14. Final Rule

MarketplaceService is the financial backbone of the experience.
Keep it simple, server-authoritative, and idempotent.

