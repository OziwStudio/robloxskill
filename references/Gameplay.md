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

## Activate When

- User asks gameplay system, combat, RPG, simulator, tycoon, obby, quest, shop, inventory, equipment, boss, class, skill tree, or progression.

## Standards

- Read `references/ImplementationPatterns.md` when gameplay loop, combat, or round skeleton code is needed.
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

## Anti-Patterns

- Client damage.
- Client reward.
- Infinite inflation.
- Duplicate quest reward.
- Hardcoded item logic.
- Unbounded grind.
- Mixed UI and rules.

## Output Focus

- Define loop first.
- Define authority second.
- Define data third.
- Keep genre-specific logic modular.
- Include first playable skeleton before extra complexity.

## Practical Principles

- First playable loop beats perfect architecture.
- Reward must follow action quickly.
- Server validates gameplay result.
- Client owns feel, animation, VFX, and local prediction only.
- Economy needs source, sink, pacing, and abuse prevention.

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

### Extra Standards

- Read the gameplay loop and round skeleton patterns before coding complex systems.
- Server owns every meaningful outcome.
- Presentation must stay separate from rules.
- Keep the core loop visible in the first minute of play.
- Balance source and sink so rewards do not inflate forever.
- Keep item identifiers stable across updates.
- Use legacy Humanoid fallback unless the modern avatar path is confirmed.

### Practical Rules

- Define the primary loop before extra features.
- Define progression before cosmetics.
- Define the first play session before late-game tuning.
- Validate distance, state, and cooldown before damage.
- Make reward grants idempotent.
- Keep genre logic modular.

### Extra Output Focus

- Show the loop.
- Show server authority.
- Show reward and sink.
- Show first playable skeleton.
