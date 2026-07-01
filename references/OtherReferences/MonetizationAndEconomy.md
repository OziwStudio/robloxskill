# Monetization And Economy

## Purpose

- Use for GamePass, Developer Product, Premium, rewarded ads, pricing, economy, shop, and purchase audit.
- Security and ethics are mandatory.

## Server Rule

- Client may prompt purchase.
- Server verifies ownership.
- Server grants reward.
- Server records receipt.
- Client only displays result.

## Product Types

| Type | Use |
|---|---|
| GamePass | permanent perk |
| Developer Product | repeatable consumable |
| Premium | engagement perk |
| Rewarded ads | optional small reward when available |

## ProcessReceipt Gate

- Only one script owns `MarketplaceService.ProcessReceipt`.
- Check product id.
- Check player exists.
- Check profile loaded.
- Check duplicate receipt.
- Grant item first.
- Persist receipt.
- Return `PurchaseGranted` only when safe.
- Return `NotProcessedYet` when retry is needed.

## Economy Gate

- Server owns currency.
- Server owns inventory.
- Server owns shop price.
- Server owns grant.
- Server owns spend.
- Use integer currency.
- Validate every transaction.
- Log important purchase events.

## Pricing Gate

- Value must be clear.
- Cost must be visible.
- Purchase must be optional.
- Avoid popup spam.
- Avoid hidden chain costs.
- Avoid pay-to-win in competitive modes.
- Disclose randomized reward odds.

## Genre Monetization

- Simulator: boosts, pets, storage, cosmetics.
- Tycoon: permanent convenience, cosmetics, layout options.
- Obby: skips, cosmetics, checkpoint convenience.
- RPG: cosmetics, storage, convenience, battle pass style rewards.
- Horror: cosmetics, revive-like options with care.
- Battle royale: cosmetics only for competitive integrity.

## Anti-Patterns

- Client-side grant.
- Missing idempotency.
- Unknown product ignored forever.
- Receipt returned granted before real grant.
- Re-prompt spam.
- Misleading product description.
- Competitive pay-to-win.

## Output Rule

- For `!monetcheck`, lead with receipt safety.
- For shop generation, include fairness and prompt cooldown.
- For economy generation, include source, sink, and inflation control.
