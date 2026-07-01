# Combat
> Roblox combat reference for server authority, hit registration, damage, cooldowns, and anti-exploit combat flow.
> Use this file as the single combat guide for AI reasoning.

## 1. Purpose

Combat is a server-authoritative system.
The client sends intent, the server validates and applies outcome, and the client renders feedback.

Use it for:
- melee combat
- ranged combat
- PvP
- PvE
- blocking and parrying
- combo systems
- cooldown enforcement

Do not use client-side combat results as truth.

## 1.1 Scope

- melee attacks
- ranged attacks
- hitboxes
- combo chains
- blocking and parrying
- cooldowns
- PvP and PvE
- skill-based combat

## 1.2 Why Server Authority Matters

- clients can edit local code
- clients can fabricate remotes
- clients can spoof hit results
- clients can lie about damage or state

The server must own combat truth or the system is exploitable.

## 2. Core Principles

1. Server owns damage, health, and combat state.
2. Client owns input, animation, VFX, and UI feedback.
3. Combat must be validated before it is applied.
4. Hit detection must be consistent and bounded.
5. Cooldowns must be enforced on the server.
6. Every combat state needs a valid lifecycle.
7. Every attack needs a counterplay path.
8. Cleanup matters for every active combat connection or effect.

## 3. Combat Flow

The correct flow is:
1. player sends attack intent
2. server checks cooldown and state
3. server performs hit detection
4. server calculates damage
5. server applies result
6. client shows confirmed feedback

## 3.1 Latency Compensation

- client may play swing animation immediately
- server confirms hit separately
- client hint can be used only as a suggestion
- server must re-run validation using server state
- if the server disagrees, the client hint is discarded

## 4. Combat State

Use a state machine or equivalent guard system.

Recommended states:
- Idle
- Attacking
- Recovery
- Blocking
- Dodging
- Stunned

Rules:
- do not allow invalid state combinations
- do not attack while stunned or dead
- use recovery windows to prevent spam
- reset state cleanly on respawn or destroy

## 5. Hit Detection

Choose detection based on weapon type.

- use box or part overlap for melee
- use raycasts for hitscan or ranged combat
- deduplicate targets during one attack
- exclude the attacker from results
- limit target count per swing when needed

Rules:
- run hit detection on the server
- use server positions, not client guesses
- keep hit volumes readable and fair

## 5.1 Hitbox Sizing Guide

| Weapon Type | Range | Width | Height |
|---|---|---|---|
| Dagger / Fist | 4-5 | 4 | 5 |
| Sword | 6-8 | 5 | 6 |
| Greatsword | 8-12 | 7 | 7 |
| Spear / Polearm | 10-14 | 3 | 5 |
| AoE Slam | 8-10 | 10 | 8 |

Rules:
- position the box in front of the attacker
- offset by half the range on the forward axis
- keep the volume fair and easy to read

## 6. Damage Rules

Damage should be deterministic and easy to reason about.

Recommended formula:

`finalDamage = baseDamage * weaponMultiplier * buffs * defense * crit * combo * typeEffectiveness`

Rules:
- base damage comes from server-owned stats
- buffs and resistances are clamped
- critical hits should be defined by the server
- blocking and parry rules must be explicit
- minimum damage should be consistent with design

## 6.1 Damage Feedback

- damage numbers are client presentation only
- blocked, crit, and heal states should be visually distinct
- confirm combat results before showing the UI feedback

## 7. Cooldown Rules

Rules:
- cooldowns are tracked on the server
- the client may display timers, but not enforce them
- server silently rejects invalid spam
- one action should have one cooldown source of truth

## 8. Parry, Block, Dodge

- block reduces or negates damage based on rules
- parry is a narrow timing window, not a free cancel
- dodge grants a short safe window and must be bounded
- every defensive action needs a cooldown or cost

Rules:
- block should be held or toggled intentionally
- parry should reward timing, not spam
- dodge should create temporary invulnerability or displacement only when designed
- defensive actions must reset cleanly on respawn

## 9. Combat Networking

Use remotes only for intent and confirmed feedback.

Rules:
- client fires attack, block, dodge intent
- server validates all arguments
- server sends combat result for VFX and UI
- do not let the client report damage totals

## 10. Animation and VFX

Combat feedback should be responsive, but not authoritative.

Rules:
- animation markers are timing helpers, not damage authority
- hit VFX and sounds may play locally after server confirmation
- camera shake and combat polish stay client-side
- cleanup all temporary effects

## 10.1 Remote Pattern

- client sends attack intent only
- server validates cooldown, state, range, ownership
- server sends combat result to clients
- clients play animation, particles, sound, and camera feedback

## 11. Performance Rules

- avoid unbounded hit checks
- keep overlap queries tight
- reuse params when possible
- limit hit targets per action
- avoid heavy loops inside per-frame combat handlers
- pool damage indicators and short-lived effects
- prefer targeted queries over full-world scans

## 12. Security Rules

Never trust the client for:
- damage
- hit targets
- cooldown completion
- stun removal
- parry success
- resource changes

Server must validate:
- player identity
- combat state
- range
- ownership
- cooldown
- target validity

## 12.1 Anti-Exploit Rules

- do not trust client hitbox results
- do not trust client damage values
- do not trust client cooldown state
- do not trust client parry claims
- silently reject invalid requests

## 13. Common Failures

### 13.1 Client-side damage

Problem: exploiter fabricates damage.
Fix: server calculates and applies damage.

### 13.2 No cooldown enforcement

Problem: attack spam.
Fix: server cooldown gate.

### 13.3 No state validation

Problem: attacking while stunned or blocked.
Fix: state machine or guard rules.

### 13.4 Unbounded hitboxes

Problem: combat hits too many targets or hits through walls.
Fix: bounded overlap, validation, and target limits.

### 13.5 Instant unavoidable attacks

Problem: no counterplay.
Fix: add telegraph, cooldown, block, dodge, or parry windows.

### 13.6 Floating point stat stacking

Problem: buffs or defense can grow without limit.
Fix: clamp all multipliers and resistances.

## 13.7 Required RemoteEvents

| RemoteEvent | Direction | Purpose |
|---|---|---|
| `AttackRemote` | Client -> Server | attack intent |
| `BlockRemote` | Client -> Server | block start/stop |
| `DodgeRemote` | Client -> Server | dodge intent |
| `CombatResultRemote` | Server -> Client | hit result, parry, VFX trigger |
| `CooldownStarted` | Server -> Client | UI cooldown sync |

## 13.8 Module Locations

| Module | Location | Purpose |
|---|---|---|
| `CombatStateMachine` | ReplicatedStorage | shared state contract |
| `DamageCalculator` | ServerScriptService | server damage logic |
| `CooldownManager` | ServerScriptService | server cooldown tracking |
| `DamageDisplay` | ReplicatedStorage | client floating damage UI |
| `MeleeCombatServer` | ServerScriptService | combat handler |
| `CombatInput` | StarterPlayerScripts | client intent relay |

## 14. Professional Behavior Rules for AI

- prioritize server authority
- keep combat readable and bounded
- separate intent from result
- use the smallest correct system shape first
- keep counters and telegraphs fair
- warn when a combat flow relies on client trust
- state the remotes and module locations when generating the system

## 15. Minimal Checklist

- server owns damage
- state is validated
- cooldowns are enforced
- hit detection is bounded
- targets are deduplicated
- client only shows feedback
- cleanup exists for combat effects
- remotes are centralized
- module boundaries are clear
- combat has counterplay

## 16. Final Rule

Combat should feel responsive to the player and trustworthy to the server.
If the client can decide the result, the combat system is broken.
