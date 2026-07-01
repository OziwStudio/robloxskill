# Roblox Monetization Systems Reference

## 1. Core Rule
All purchase granting must happen on the server. The client may prompt purchases, but it must never decide ownership or rewards.

## 2. Monetization Channels
Roblox monetization usually uses:

- GamePasses for permanent unlocks
- Developer Products for repeatable purchases
- Premium Payouts for Premium engagement revenue
- Rewarded Video Ads for opt-in ad rewards

Each channel serves a different job. Use the smallest one that fits the design.

## 3. GamePasses
GamePasses are permanent account-owned purchases.

Use them for:
- VIP perks
- permanent boosts
- cosmetic unlocks
- feature unlocks

### GamePass Rules
- check ownership on the server
- grant on join
- re-grant on respawn if needed
- re-grant immediately after mid-session purchase
- handle API failure with `pcall`
- cache confirmed ownership when possible
- never assume ownership from UI state
- make perk grants idempotent

```luau
local MarketplaceService = game:GetService("MarketplaceService")

local ok, ownsPass = pcall(function()
	return MarketplaceService:UserOwnsGamePassAsync(player.UserId, gamePassId)
end)
```

### Prompt Rule
Client may show the prompt, but the grant logic stays server-side.

### Join Flow
1. Player joins.
2. Server checks configured passes.
3. Server grants owned perks.
4. Server re-applies perks on respawn if needed.
5. Server listens for mid-session purchases.

## 4. Developer Products
Developer Products are consumable and repeatable.

Use them for:
- currency packs
- temporary boosts
- skip items
- revives
- loot crates

### ProcessReceipt Rule
`MarketplaceService.ProcessReceipt` is the critical purchase confirmation path.

- only one script may own it
- return `PurchaseGranted` only after the item is actually granted
- return `NotProcessedYet` if granting or persistence fails
- handle missing player cases
- deduplicate receipts
- use a purchase history store or equivalent idempotency guard
- keep the callback fast and deterministic
- do not perform unrelated work inside receipt handling

```luau
MarketplaceService.ProcessReceipt = function(receiptInfo)
	return Enum.ProductPurchaseDecision.NotProcessedYet
end
```

### Receipt Safety
- save receipt history before finalizing when possible
- log every successful grant
- never trust client purchase state
- never grant before validation
- retry safely when persistence is unavailable
- preserve a manual recovery path for failed grants

## 5. Premium Payouts
Premium Payouts come from time spent by Premium users.

### Premium Rules
- detect Premium on join
- react to membership changes
- give Premium perks if the design supports it
- use perks to increase retention, not pressure
- keep Premium perks visible but not mandatory
- update perks when membership changes mid-session

```luau
if player.MembershipType == Enum.MembershipType.Premium then
	player:SetAttribute("IsPremium", true)
end
```

## 6. Rewarded Video Ads
Rewarded ads should be opt-in and should not interrupt gameplay.

### Good Placement
- lobby
- waiting area
- between rounds
- revive screen
- daily bonus screen

### Bad Placement
- mid-combat
- forced popup
- blocking progression
- repeated interruption spam

### Ad Rules
- give a clear reward
- add cooldowns
- remove ad pressure if the player declines
- keep reward value meaningful but not game-breaking
- gate by cooldown and state
- do not stack ad prompts on top of other prompts
- log completions and cooldown state

## 7. Pricing Strategy
Common Roblox price bands:

- 25 Robux: smallest meaningful offer
- 50 Robux: small bundle
- 75 Robux: mid small offer
- 100 Robux: standard offer
- 250 Robux: strong offer
- 500 Robux: major offer
- 1,000 Robux: high-tier offer
- 2,500+ Robux: very high tier

### Pricing Rules
- use anchoring with the expensive item first
- use bundles with better per-unit value
- avoid pricing below the practical floor
- use odd numbers when they help conversion
- use limited-time offers carefully and honestly
- keep the first offer understandable in one glance
- show the value difference between tiers
- make the best value option easy to spot

## 8. DevEx Math
Use the current exchange rate when projecting earnings.

### Projection Formula
```text
Daily Revenue (Robux) = DAU x Conversion Rate x Average Purchase
Monthly Revenue (Robux) = Daily Revenue x 30
Monthly Revenue (USD) = Monthly Revenue (Robux) x DevEx Rate
```

### Planning Rule
- estimate DAU
- estimate conversion
- estimate average purchase
- compare against development cost
- keep projections realistic
- treat projections as estimates, not guarantees
- model both direct sales and Premium revenue

## 9. Ethical Monetization
Monetization must be fair.

### Do
- give real value
- disclose randomized odds
- keep core gameplay free
- show price clearly
- offer earnable alternatives when possible
- respect declined prompts
- make monetization feel additive, not coercive
- let players understand the offer before the prompt

### Do Not
- do not make competitive pay-to-win
- do not hide total cost
- do not use fake scarcity
- do not pressure children
- do not block core progression behind payment
- do not misdescribe the product

## 10. Purchase Verification
Always verify on the server.

### Safe Pattern
- prompt on client
- verify on server
- grant on server
- persist if needed
- log the result
- keep prompt and grant separated
- never infer success from the prompt closing

```luau
local success, result = pcall(function()
	return MarketplaceService:UserOwnsGamePassAsync(player.UserId, passId)
end)
```

### Verification Flow
1. Client prompts purchase.
2. Roblox confirms the purchase.
3. Server receives the callback.
4. Server validates the player and product.
5. Server grants the reward.
6. Server records the outcome.

## 11. Logging
Log purchase events for support and debugging.

- player id
- product or pass id
- receipt id
- reward granted
- failure reason
- timestamp
- player name
- grant result

## 12. Studio Testing
- test prompts in Studio
- test ownership checks
- test receipt handling
- test rejoin behavior
- test failure paths
- test duplicate callback safety
- test player-leaves-before-grant path
- test persistence failure path
- test cooldown behavior for ad prompts

## 13. Anti-Patterns

- client-side reward granting
- returning `PurchaseGranted` too early
- missing idempotency
- no error handling
- aggressive popup spam
- misleading descriptions
- hidden total cost
- treating UI as proof of ownership
- putting multiple scripts on `ProcessReceipt`
- granting without persistence when persistence is required

## 14. Implementation Shape
A solid monetization system should keep these parts separate:

- channel config
- prompt logic
- server verification
- grant handlers
- persistence
- logging
- cooldowns
- ethics guardrails
- receipt store
- membership hooks
- reward policy
- test hooks

## 15. Output Focus
When generating monetization code:

- keep the grant path server-side
- keep receipts idempotent
- keep prompts minimal
- keep rewards clear
- keep the implementation easy to audit
- keep purchase handling short and explicit
- keep error paths visible
- keep prompt logic and grant logic separate
