# Game Design

## Purpose

- Use for core loop, FTUE, retention, social design, difficulty, updates, launch planning, and full game scoping.
- Use when the user asks for a new game, a complete system, a genre plan, publish prep, or a broad game audit.
- Make the first playable fast.
- Keep server authority from day one.
- Keep data safe from day one.
- Keep monetization optional until the game has value.
- Treat design as a retention system, not just a feature list.
- Design for social discovery, mobile play, and short sessions first.

## Complete Game Stack

1. Core loop.
2. FTUE.
3. Progression.
4. Meta-progression.
5. Social surface.
6. Data persistence.
7. Server authority.
8. Networking.
9. UI/UX.
10. Monetization.
11. Performance.
12. Testing.
13. Publish readiness.
14. Update strategy.

## Roblox Reality

- Mobile is first-class.
- Social discovery matters.
- Short sessions need fast reward.
- Retention drives growth.
- Monetization must feel fair.
- Players discover through friends, home page, and recommendations.
- Players leave fast if the first minute is weak.
- The game must work on low-end and mid-range mobile devices.

## Loop Stack

| Loop | Time | Goal |
|---|---|---|
| Primary loop | 5-30 seconds | satisfying action |
| Progression | 10-30 minutes | session goal |
| Meta-progression | days-months | return reason |

## Core Loop Design

The primary loop must feel good without progress context.

Rules:
- give the player something to do immediately
- give visual and audio reward on every meaningful action
- keep the cadence fast
- do not leave the player idle for too long

## Progression Engine

The progression layer must create a meaningful milestone within a single session.

Patterns:
- earn currency and upgrade tools
- unlock zones with concrete value
- acquire equipment or pet tiers
- unlock abilities or builds

Rules:
- make progress visible
- make each milestone feel like a step forward
- do not force long sessions without reward

## Meta-Progression

The long-term layer keeps players returning.

Patterns:
- account-level unlocks
- collection completion
- prestige or rebirth
- social investment
- seasonal exclusives

Rules:
- long-term goals should outlive one session
- collection progress should be visible
- prestige should feel like a new path, not a punishment

## Loop Flywheel

Primary loop feeds progression.
Progression feeds meta-progression.
Meta-progression improves the primary loop.

If one layer is missing, the game loses momentum.

## FTUE Gate

- First action within 5 seconds.
- First reward within 15-30 seconds.
- First clear objective visible.
- No forced text wall.
- No shop spam on join.
- Show one system at a time.
- Teach by doing.
- Use visual guidance before text.
- Let the player learn the core loop within the first minute.
- Keep the first minute low-friction and high-reward.

## FTUE Rules

- first action within 5 seconds
- first reward within 15 seconds
- first wow moment within 30 seconds
- no hard tutorial wall
- no forced long explanation
- no confusion about the next objective

## First-Time User Experience

The player should understand the game without needing a manual.

Rules:
- show the first objective clearly
- reveal new systems progressively
- let failure be safe early on
- give a visible reward path immediately
- make the first minute feel exciting

## Progressive Disclosure

- Wave 1: core loop only
- Wave 2: first upgrade path
- Wave 3: social and shop systems
- Wave 4: advanced systems and long-term goals

Do not show everything at once.

## Retention Gate

- Daily reason to return.
- Visible collection progress.
- Social surface.
- Event or update cadence.
- Progress in short sessions.
- Long-term prestige or mastery.
- Daily login rewards should feel worthwhile.
- Limited-time events should create urgency without permanent power loss.
- Social systems should make the game harder to leave.

## Retention Mechanics

### Daily Rewards

- small early rewards
- meaningful mid-week rewards
- strong weekly anchor reward
- visible reward calendar

Rules:
- do not punish a missed day too harshly
- show the reward track
- make collection feel ceremonial

### Streak Systems

- login streaks
- play streaks
- quest streaks
- win streaks

Rules:
- streaks should motivate, not humiliate
- streak values should increase in a readable way
- streak breaks should not destroy the entire system

### Limited-Time Events

- seasonal events
- weekend events
- collaboration events

Rules:
- time-limited cosmetics are fine
- time-limited power is not
- communicate event timing clearly

### Collection Completion

- visible progress bars
- rarity tiers
- completion bonuses
- browsable collection UI

Rules:
- make missing items visible
- let completion create a strong “one more item” pull
- give collections long-term meaning

## Social Gate

- Friends can play together.
- Players can show progress.
- Leaderboard or social comparison exists when suitable.
- Trading has scam prevention.
- Competitive features stay fair.
- Players should have something to show off.
- Social systems should create reason to return with friends.

## Social Systems

### Guilds and Groups

- low barrier to entry
- shared goals
- group progression
- group identity

### Leaderboards

- daily, weekly, all-time, and friend-only boards
- multiple metrics for different player types
- visible but not oppressive
- anti-cheat integrity matters

### Cooperative Mechanics

- team dungeons or raids
- shared goals
- mentorship rewards
- asymmetric cooperation

### Trading

- trading should create social glue
- value estimates and confirmation screens matter
- scam prevention must be built in

## Difficulty Gate

- Difficulty ramps gradually.
- Failure reason is clear.
- Repeated failure gets hint or alternative.
- Skill is rewarded.
- Grind has novelty.
- Mobile controls are respected.
- Failure should teach.
- Challenge should be understandable.
- Difficulty should feel fair, not deceptive.

## Difficulty Design

### Difficulty Curve

- gradual ramp
- occasional plateau
- boss spike only when trained

### Skill vs Time

- skill-heavy games reward mastery
- time-heavy games reward persistence
- hybrid games use skill to speed progression

### Avoiding Frustration

- clear failure feedback
- hints after repeated failure
- optional skip when appropriate
- adjustable difficulty when the genre allows it

### Rewarding Mastery

- speed records
- perfect run bonuses
- style or combo systems
- visible mastery indicators

## Update Strategy

- Critical bugs first.
- Retention features second.
- FTUE improvements third.
- New content fourth.
- Monetization after game value is clear.
- Updates should be planned, communicated, and measurable.

## LiveOps

### Content Cadence

- weekly small updates
- monthly feature updates
- quarterly major updates

### Seasonal Planning

- plan early
- build and test before hype
- launch with monitoring
- review after event ends

### Feedback Loops

- in-game feedback
- community channels
- analytics
- creator feedback

### Priority Order

1. critical bugs and exploits
2. retention features
3. FTUE improvements
4. content additions
5. monetization

## Genre Playbooks

### Simulator

- Core loop: action, reward, upgrade, unlock.
- Protect reward calculation on server.
- Add rate limits to action remotes.
- Separate temporary boosts from saved data.
- Keep UI responsive on mobile.

### Tycoon

- Server owns purchases, droppers, and income.
- Client only requests purchase intent.
- Validate plot ownership.
- Persist unlocked items by id.
- Avoid hundreds of active loops per plot.

### Obby

- Server validates checkpoint order.
- Save checkpoint safely.
- Avoid client-only wins.
- Use touch debounce.
- Keep parts mobile-friendly.

### RPG

- Server owns stats, combat, drops, and quests.
- Validate enemy distance and state.
- Use deterministic reward grants.
- Version item schemas.
- Avoid duplicated quest rewards.

### Horror

- Server owns critical scares, enemy state, and win conditions.
- Client owns presentation effects only.
- Keep audio and visual effects bounded.
- Avoid forcing high graphics settings.

### Battle Royale

- Server owns match state, inventory, damage, zone, and rewards.
- Validate weapon fire intent.
- Keep replication scoped.
- Use clear round lifecycle.
- Prevent reconnect reward abuse.

### Social Hub

- Server owns permissions and moderation-related actions.
- Client owns cosmetic UI.
- Validate teleport, emote, trade, and invite actions.
- Add abuse cooldowns.

### Admin System

- Ask user for owner `UserId`.
- Do not inject default owner id.
- Server validates rank and command permission.
- Log dangerous commands.
- Avoid client-side admin authority.

### General Design Rules by Genre

#### Simulator

- action, reward, upgrade, unlock
- visible power growth
- prestige/rebirth when ceiling is reached
- avoid monotony by adding variety

#### Tycoon

- build, earn, expand, automate
- visible construction progress
- choice and customization matter
- do not make it only linear pad stepping

#### Obby

- attempt, fail, learn, succeed
- checkpoint generosity
- fair difficulty
- visual variety every few stages

#### RPG

- quest, fight, loot, level, explore
- character identity matters
- loot anticipation matters
- keep tutorial short and mobile-friendly

#### Horror

- explore, tension, scare, survive
- atmosphere first
- pacing matters
- replayability through mystery

#### Battle Royale

- drop, loot, fight, survive
- fast matches
- skill expression
- spectating keeps social groups engaged

## Best Practices

- test with real players early
- use analytics as design feedback
- balance depth with accessibility
- respect player time
- make the first minute amazing

## Anti-Patterns

- infinite grind with no variety
- pay-to-win
- neglecting mobile
- no social features
- copying without understanding

## Design Checklist

- core loop is satisfying within 10 seconds
- first reward happens within 30 seconds
- the player knows what to do without reading much
- mobile experience is functional
- social feature exists
- session delivers progress
- there is a reason to return tomorrow
- monetization is optional
- performance is acceptable on mobile
- no dead-end state exists

## Intent Router

| Intent | Read |
|---|---|
| New game | `references/bridges/WorkflowPlaybooks.md`, `references/bridges/TemplateBridge.md`, `references/topics/MonetizationAndEconomy.md`, `references/Production.md`, `references/topics/TestingQA.md` |
| Complete game | `references/bridges/WorkflowPlaybooks.md`, `references/bridges/TemplateBridge.md`, `references/Performance.md`, `references/Architecture.md`, `references/Networking.md`, `references/Data.md` |
| Monetization | `references/topics/MonetizationAndEconomy.md`, `references/Production.md`, `references/Data.md`, `references/Security.md` |
| Testing | `references/topics/TestingQA.md`, `references/bridges/WorkflowPlaybooks.md` |
| Studio automation | `references/StudioModes.md` |
| Publish | `references/bridges/WorkflowPlaybooks.md`, `references/Production.md`, `references/topics/TestingQA.md` |

## New Game Order

1. Detect genre.
2. Load template bridge.
3. Define scope: MVP, standard, full, live-service.
4. Define file tree.
5. Generate scaffold.
6. Generate core systems.
7. Add data and remotes.
8. Add UI.
9. Add monetization only after fairness.
10. Add tests and playtest checklist.
11. Run production gates.

## Complete Game Gate

- Core loop works in 10 seconds.
- First reward happens within 30 seconds.
- Player knows next action.
- Mobile is playable.
- Social surface exists.
- Data cannot be lost by normal leave/rejoin.
- Remotes cannot grant value directly.
- Purchases are server-granted.
- Publish checklist passes.

## Output Rule

- For game design, state loop stack first.
- For new game, state first 60 seconds.
- For retention, state D1/D7/D30 intent.
- For monetization, state fairness rule.
