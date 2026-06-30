# Gameplay

## Rules

- Server owns damage, rewards, inventory, rank, currency, and cooldown.
- Client owns camera, input, local animation, visual feedback, and UI.
- Validate distance for touch, pickup, trade, combat, carry, and interact systems.
- Use attributes for simple replicated state.
- Use modules for reusable gameplay logic.
- Use configs for tunable values.

## Common Systems

- Carry: server validates target, distance, cooldown, alive state, and release.
- Dance: client plays animation; server may replicate emote state.
- Shop: server validates item, price, stock, ownership, and currency.
- Donation: server validates product receipt and grants reward once.
- Relationship: server validates request, consent, cooldown, and persistence.
- Combat: server validates weapon, range, state, team, cooldown, and damage.

## State Names

- Use short explicit states: `Idle`, `Busy`, `Trading`, `Carrying`, `Stunned`, `Dead`.
- Do not store gameplay truth only in UI.
- Clear state on death, leaving, teleport, or system shutdown.
