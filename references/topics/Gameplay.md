# Gameplay Specialist

## Scope

- Combat.
- Skills.
- Hitbox.
- Status effects.
- RPG stats.
- Progression.
- Inventory.
- Economy.
- Quest.
- Reward loop.
- Retention.
- Grind balance.
- Reward structure.
- Boss systems.
- Modern Roblox gameplay.
- Multiplayer loop.
- Lobby.
- Teams.
- Round state.
- Matchmaking.
- Teleport flow.
- Server lifecycle.
- Queue.
- Server browser.
- Reserved server.
- Private server.
- Spawn flow.
- Respawn control.
- Session handoff.

## Activate When

- User asks gameplay system, combat, RPG, simulator, tycoon, obby, quest, shop, inventory, equipment, boss, class, skill tree, or progression.

## Standards

- Read `references/bridges/ImplementationPatterns.md` when gameplay loop, combat, or round skeleton code is needed.
- Use `Server Authority Gameplay` and `Round State Machine` patterns.
- Server owns gameplay outcomes.
- Separate rules from presentation.
- Keep progression loop clear.
- Balance reward source and sink.
- Validate combat range and state.
- Prevent duplicate rewards.
- Keep item ids stable.
- Keep systems extensible.
- Keep legacy Humanoid fallback unless modern avatar system is confirmed.
- Validate modern interaction APIs before relying on them.
- Define primary loop, progression loop, and meta-progression for full games.
- Define first 60 seconds for new games.
- Evaluate retention, grind pacing, and reward pacing.
- Build for long-term player return, not only first-session novelty.
- Keep multiplayer state server-owned.
- Keep lobby state explicit.
- Keep round transitions deterministic.
- Keep team assignment server-side.
- Keep teleport and cross-server flow separate from combat.
- Keep queue state server-owned.
- Keep reserved and private server flow explicit.
- Keep spawn and respawn rules separate.

## Gameplay Gate

- What is the core loop?
- What is the reward?
- What is the sink?
- What is saved?
- What is temporary?
- What can be exploited?
- What grows over time?
- What modern API needs fallback?
- What makes player return tomorrow?
- Is retention strong enough?
- Is grind pacing fair?
- Is reward pacing healthy?
- How does the lobby start?
- How does the round end?
- How do players rejoin flow?
- What happens when a player leaves mid-round?
- How does the queue fill?
- How does the server choose a match?
- How does a reserved server receive players?
- What happens on respawn?

## Anti-Patterns

- Client damage.
- Client reward.
- Infinite inflation.
- Duplicate quest reward.
- Hardcoded item logic.
- Unbounded grind.
- Mixed UI and rules.
- Client-controlled lobby state.
- Client-controlled team assignment.
- Round logic without state machine.
- Server cleanup missing on leave.
- Teleport flow mixed into combat code.
- Queue logic mixed with UI logic.
- Server browser data trusted from client.
- Respawn logic not reset after round end.
- Reward pacing that inflates forever.
- Grind that feels unbounded.
- Boss flow without server authority.

## Output Focus

- Define loop first.
- Define authority second.
- Define data third.
- Keep genre-specific logic modular.
- Include first playable skeleton before extra complexity.
- Include lobby, round, and result flow when multiplayer exists.
- Include server entry and exit behavior when players join or leave.
- Include queue, match select, and handoff flow when cross-server play exists.
- Include retention, grind balance, and reward structure when tuning progression.

## Practical Principles

- First playable loop beats perfect architecture.
- Reward must follow action quickly.
- Server validates gameplay result.
- Client owns feel, animation, VFX, and local prediction only.
- Economy needs source, sink, pacing, and abuse prevention.
- Multiplayer needs join, ready, start, play, results, and reset flow.
- Team systems should be simple and server-owned.
- Round games need manual respawn control when the mode depends on it.
- Cross-server games need clear handoff and arrival rules.
- Queue systems should be server-side and rate-limited.
- Server browser data should be read-only on the client.
- Retention should come from loop quality, not only rewards.
- Reward structure should support source, sink, pacing, and abuse prevention.

## Loop Contract Example

```text
Action: player defeats enemy
Server Validation: alive, range, cooldown, target valid
Reward: XP + possible item drop
Sink: repair, crafting, upgrade cost
Saved: XP, level, inventory ids
Client Feedback: hit marker, sound, VFX
Abuse Case: fake hit remote
```

## Player Lifecycle

- `PlayerAdded` starts player setup.
- `CharacterAdded` handles spawn setup.
- `CharacterRemoving` handles cleanup before character loss.
- `PlayerRemoving` saves and clears state.
- `BindToClose` acts as shutdown fallback.

```luau
local Players = game:GetService("Players")

local function onPlayerAdded(player: Player)
	print(player.Name)
end

Players.PlayerAdded:Connect(onPlayerAdded)
for _, player in Players:GetPlayers() do
	task.spawn(onPlayerAdded, player)
end
```

## Team Systems

- Keep team assignment on the server.
- Use teams for spawn, UI, and damage rules.
- Prevent friendly fire when the mode requires it.
- Keep lobby players separate from active players.

```luau
local function canDamage(attacker: Player, victim: Player): boolean
	if attacker.Team == victim.Team then
		return false
	end

	return true
end
```

## Boss Systems

- Bosses need server-owned health and phases.
- Boss attacks need clear telegraph and authority.
- Boss rewards need anti-duplication checks.
- Boss pacing should support the progression loop.

## Progression Rules

- Define primary loop first.
- Define progression loop second.
- Define meta-progression third.
- Keep item identifiers stable across updates.
- Keep level, unlock, and reward pacing coherent.
- Keep reward gains quick enough to feel good.
- Keep sinks strong enough to avoid inflation.

## Retention Rules

- Build a first 60 seconds that is understandable.
- Build a loop that rewards the action quickly.
- Build a loop that gives players a reason to return.
- Keep early progression readable.
- Keep later progression from becoming empty grind.

## Grind Rules

- Avoid unbounded grind.
- Keep grind pacing intentional.
- Use source and sink to balance inflation.
- Keep progression speed aligned with reward structure.
- Avoid reward curves that break the economy.

## Reward Structure

- Reward source must be clear.
- Reward sink must be clear.
- Reward pacing must be clear.
- Reward grants must be idempotent.
- Reward values must come from server-owned logic.

## Spawn Flow

- Pick spawn by team or lobby state.
- Keep spawn choice server-owned.
- Reset spawn rules when the round changes.
- Use manual respawn when the game mode needs it.

## Respawn Control

- Disable auto spawn when the round system needs control.
- Respawn only when the match state allows it.
- Reapply loadout and team state after respawn.
- Rebuild transient character state on every spawn.

## Queue Flow

- Queue joins must be server-validated.
- Queue position should not be client-authoritative.
- Start match only when the queue condition is valid.
- Remove players from queue on leave or cancel.

## Server Browser

- Server browser data should describe public state only.
- Do not trust client-picked server ids without validation.
- Hide invalid or full servers from the join flow.
- Refresh listings on a controlled interval.

## Reserved Server Flow

- Use reserved servers for controlled sessions.
- Pass arrival metadata only when needed.
- Validate access before teleporting.
- Treat reserved server ids as server-owned state.

## Private Server Flow

- Private server access should follow game rules.
- Do not bypass gameplay validation because a server is private.
- Keep owner-only behavior separate from match rules.
- Keep session permissions explicit.

## Session Handoff

- Handoff should preserve only required state.
- Restore match context on arrival.
- Rebuild transient state after teleport.
- Never store live gameplay objects in teleport payloads.

## Lobby Flow

- Track ready state on the server.
- Broadcast lobby status to clients.
- Start the round only when the minimum player rule is met.
- Use countdown before the round starts.
- Reset lobby state after round end.
- Use ready toggles or queue joins, not client-owned start flags.
- Clear ready state when players leave.
- Reuse lobby state after every match.

## Round State Machine

Use a state machine for production round games:

`Intermission -> Countdown -> Playing -> Results -> Intermission`

- Intermission prepares the next round.
- Countdown warns players before start.
- Playing handles the active match.
- Results shows outcome and scores.

```luau
export type RoundState = "Intermission" | "Countdown" | "Playing" | "Results"
```

## Round Rules

- Broadcast state changes.
- Keep timer values server-owned.
- Recheck player count during countdown.
- Abort start if the lobby becomes invalid.
- Reset participants after each round.
- Handle join and leave during any phase.
- Rebuild teams and spawn state when the state machine changes.
- End the round cleanly before the next queue starts.

## Matchmaking

- Matchmaking should select the right lobby or server state.
- Keep queue logic server-side.
- Avoid direct client selection of match result.
- Use simple balancing before advanced ranking.

## Teleport Flow

- Use TeleportService for multi-place flow.
- Pass only required metadata.
- Validate teleport payloads on arrival.
- Keep reserved or private server flow explicit.
- Treat teleport as a handoff, not as state storage.

## Server Lifecycle

- Load data on join.
- Prepare gameplay state on spawn.
- Save on leave.
- Recover on shutdown.
- Rebuild transient state after reconnect.

## Multiplayer Output Focus

- Show player lifecycle.
- Show team ownership.
- Show lobby start and reset.
- Show round state transitions.
- Show server validation for competitive actions.
- Show queue and handoff when the game spans servers.
- Show respawn and spawn rules when the mode needs control.

## Combat Skeleton

```luau
local function canDamage(player: Player, target: Model): boolean
	local character = player.Character
	if not character then
		return false
	end

	local root = character:FindFirstChild("HumanoidRootPart")
	local targetRoot = target:FindFirstChild("HumanoidRootPart")
	if not root or not targetRoot then
		return false
	end

	return (root.Position - targetRoot.Position).Magnitude <= 12
end
```

## Specialist Habit

- Define action, reward, sink, save, and abuse case before code.
- Keep visual feedback separate from server outcome.
- Make the first 60 seconds understandable.
- In multiplayer, define lobby, round, team, and leave behavior before code.
- In cross-server games, define queue, browser, teleport, and arrival behavior before code.

## Deepin Merge

### Scope Additions

- Combat flow.
- Hit validation.
- Status effects.
- RPG stats.
- Progression layers.
- Inventory rules.
- Quest loops.
- Reward pacing.
- Lobby control.
- Team balance.
- Round transitions.
- Matchmaking.
- Teleport handoff.
- Player lifecycle.
- Queue handling.
- Server browsing.
- Reserved/private server flow.
- Spawn control.
- Respawn control.
- Session handoff.
- Retention loop.
- Grind pacing.
- Reward pacing.
- Boss flow.

### Extra Standards

- Read the gameplay loop and round skeleton patterns before coding complex systems.
- Server owns every meaningful outcome.
- Presentation must stay separate from rules.
- Keep the core loop visible in the first minute of play.
- Balance source and sink so rewards do not inflate forever.
- Keep item identifiers stable across updates.
- Use legacy Humanoid fallback unless the modern avatar path is confirmed.
- Keep multiplayer state explicit and resettable.
- Use a state machine for match flow.
- Keep lobby status separate from round status.
- Keep TeleportService separate from gameplay rules.
- Keep queue logic separate from lobby UI.
- Keep server browser data read-only.
- Keep reserved server access explicit.
- Keep private server access explicit.
- Keep progression readable and scalable.
- Keep rewards fair and predictable.
- Keep boss systems server authoritative.

### Practical Rules

- Define the primary loop before extra features.
- Define progression before cosmetics.
- Define the first play session before late-game tuning.
- Validate distance, state, and cooldown before damage.
- Make reward grants idempotent.
- Keep genre logic modular.
- Define ready flow before round flow.
- Define team flow before damage flow.
- Define leave handling before match start.
- Define queue flow before server handoff.
- Define respawn flow before reward flow in round games.
- Define retention loop before late-game tuning.
- Define grind balance before scaling content.

### Extra Output Focus

- Show the loop.
- Show server authority.
- Show reward and sink.
- Show first playable skeleton.
- Show lobby and round state.
- Show join, leave, and reset behavior.
- Show queue, browser, teleport, and arrival behavior.
- Show retention, grind balance, and reward structure.
