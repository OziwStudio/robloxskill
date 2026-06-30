# Gameplay

## Purpose

- Pakai untuk combat, progression, inventory, economy, quest, round, genre, and game loop.
- Pakai `ImplementationPatterns.md` untuk combat authority dan round state machine.

## Core Loop

- Define action.
- Define reward.
- Define upgrade.
- Define unlock.
- Define return reason.

## Loop Contract

```text
Action:
Server Validation:
Reward:
Cost/Sink:
Progress Saved:
Client Feedback:
Abuse Case:
Failure State:
```

## Progression

- Primary loop must feel good immediately.
- Session progression must give a milestone in 10-30 minutes.
- Meta-progression must create return value.
- First reward should happen fast.

## Systems

- Combat: server validates hit, range, cooldown, state, and damage.
- Inventory: server owns item instances and transactions.
- Economy: define source, sink, inflation control.
- Quest: prevent duplicate rewards.
- Round: use explicit state machine.
- Social: add visible progress, friend play, or leaderboard.

## Combat Pattern

- Client sends attack intent only.
- Server checks cooldown, alive state, weapon, target, distance, line-of-sight when needed.
- Server computes damage.
- Server applies damage.
- Client receives VFX/SFX feedback.
- Read `AnimationVFX.md` for animation markers, particles, camera shake, and visual feedback rules.

## Economy Pattern

- Source creates currency.
- Sink removes currency.
- Server owns both.
- UI displays values only.
- Purchases call server-owned product config.

## Quest Pattern

- Server tracks objective progress.
- Reward grants once.
- Completion stores quest id or state.
- Rejoining must not duplicate reward.

## Genre Hooks

- Simulator: collect, upgrade, unlock, prestige.
- Tycoon: build, earn, expand, automate.
- Obby: attempt, fail, learn, succeed.
- RPG: quest, fight, loot, level, explore.
- Horror: explore, tension, scare, survive.
- Battle royale: drop, loot, fight, survive.

## Anti-Patterns

- Client damage.
- Client reward.
- No clear next objective.
- Infinite grind without novelty.
- Pay-to-win competitive advantage.
- Mobile-unfriendly controls.

## Concrete Patterns

- Combat authority: `ImplementationPatterns.md#Server Authority Gameplay`.
- Round manager: `ImplementationPatterns.md#Round State Machine`.
- Receipt reward: `ImplementationPatterns.md#Receipt Idempotency`.
