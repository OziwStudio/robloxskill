# Genre Playbooks

## Simulator

- Core loop: action, reward, upgrade, unlock.
- Protect reward calculation on server.
- Add rate limits to action remotes.
- Separate temporary boosts from saved data.
- Keep UI responsive on mobile.

## Tycoon

- Server owns purchases, droppers, and income.
- Client only requests purchase intent.
- Validate plot ownership.
- Persist unlocked items by id.
- Avoid hundreds of active loops per plot.

## Obby

- Server validates checkpoint order.
- Save checkpoint safely.
- Avoid client-only wins.
- Use touch debounce.
- Keep parts mobile-friendly.

## RPG

- Server owns stats, combat, drops, and quests.
- Validate enemy distance and state.
- Use deterministic reward grants.
- Version item schemas.
- Avoid duplicated quest rewards.

## Horror

- Server owns critical scares, enemy state, and win conditions.
- Client owns presentation effects only.
- Keep audio and visual effects bounded.
- Avoid forcing high graphics settings.

## Battle Royale

- Server owns match state, inventory, damage, zone, and rewards.
- Validate weapon fire intent.
- Keep replication scoped.
- Use clear round lifecycle.
- Prevent reconnect reward abuse.

## Social Hub

- Server owns permissions and moderation-related actions.
- Client owns cosmetic UI.
- Validate teleport, emote, trade, and invite actions.
- Add abuse cooldowns.

## Admin System

- Ask user for owner `UserId`.
- Do not inject default owner id.
- Server validates rank and command permission.
- Log dangerous commands.
- Avoid client-side admin authority.
