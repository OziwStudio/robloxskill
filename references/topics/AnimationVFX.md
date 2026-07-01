# Animation And VFX

## Purpose
- Use for animation, VFX, SFX timing, particles, tweens, camera shake, cutscenes, abilities, combat feedback, and visual polish.
- Read with `references/Performance.md` when effects may affect FPS, memory, mobile, or network replication.
- Read with `Gameplay.md` when VFX represents gameplay feedback.
- Read with `Security.md` when client VFX could be confused with server-owned outcomes.

## Priority
1. Gameplay clarity.
2. Performance safety.
3. Cleanup.
4. Network economy.
5. Visual polish.

## Scope Additions
- Character animation
- NPC animation
- Particle effects
- Beams and trails
- Tween-driven feedback
- Lighting and post-processing
- Sound timing
- Camera effects
- Cutscenes
- General visual polish

## Animation Rules
- Server owns gameplay result.
- Client may play local animation feedback.
- Do not let animation events grant damage, reward, currency, or inventory.
- Use animation markers for timing feedback only unless server validates the outcome.
- Keep combat hit confirmation server-side.
- Stop or replace animation tracks when state changes.
- Clean character animation connections on respawn.
- Keep legacy Humanoid fallback unless modern avatar controller is confirmed.
- Prefer `Animator` for Humanoid rigs.
- Use `AnimationController` for non-Humanoid rigs.
- Pick animation priority deliberately.
- Cross-fade when one track replaces another.
- Use markers for hit timing, not authority.

## VFX Rules
- VFX is presentation, not authority.
- Spawn frequent VFX locally when possible.
- Replicate only the minimum event needed to show VFX.
- Pool repeated effects.
- Destroy one-shot emitters, beams, trails, sounds, and attachments.
- Bound particles by lifetime, rate, size, and texture cost.
- Disable or simplify heavy VFX on low-end devices.
- Keep mobile and crowded servers first-class.
- Use `TweenService` for simple visual motion.
- Use `Beam` for links, lasers, and tethers.
- Use `Trail` for motion streaks.
- Use `ParticleEmitter` bursts instead of endless spawning.
- Use lighting effects sparingly and clean them up.

## Remote Pattern
```text
Client -> Server:
  Ability_Cast_RE { AbilityId, AimPosition }

Server:
  validate cooldown, state, range, ownership
  compute result
  apply damage/reward/state

Server -> Clients:
  Ability_FX_RE { AbilityId, Origin, Target, ResultId }

Clients:
  play animation, particles, sound, camera feedback
```

## Effect Layers

- Animation layer: pose, motion, timing.
- VFX layer: particles, beams, trails, lights.
- SFX layer: impact, movement, feedback.
- Camera layer: shake, zoom, focus.
- Gameplay layer: server authority and result.

## Effect Budget
| Effect | Rule |
|---|---|
| Particles | bound rate and lifetime |
| Trails | disable after action ends |
| Beams | destroy or pool after use |
| Sounds | destroy one-shot sounds |
| Camera shake | restore camera and disconnect |
| Tweens | cancel or complete before destroy |
| Highlight | limit count and duration |
| Lights | avoid shadow-heavy spam |
| Animation tracks | stop or replace cleanly |
| Markers | timing only, not authority |

## Cleanup Pattern
```luau
--!strict

local Debris = game:GetService("Debris")

local function emitOnce(template: Instance, parent: Instance, lifetime: number): Instance
	local effect = template:Clone()
	effect.Parent = parent

	for _, descendant in effect:GetDescendants() do
		if descendant:IsA("ParticleEmitter") then
			descendant:Emit(descendant:GetAttribute("BurstCount") or 10)
		end
	end

	Debris:AddItem(effect, lifetime)
	return effect
end
```

## Animation Marker Pattern
```luau
--!strict

local function bindMarker(track: AnimationTrack, markerName: string, callback: () -> ()): RBXScriptConnection
	return track:GetMarkerReachedSignal(markerName):Connect(function()
		callback()
	end)
end
```

## Camera Shake Rule
- Camera shake must be local.
- Camera shake must have duration.
- Camera shake must not permanently change camera mode.
- Disconnect render-step bindings after effect ends.
- Reduce shake on mobile and accessibility-sensitive modes.

## Sound Sync Rule
- Sync hit sounds to marker or collision timing.
- Keep one-shot sounds short-lived.
- Destroy temporary sounds after they end.
- Match playback speed to animation speed when needed.

## Mental Model
- Server decides what happened.
- Client shows what it felt like.
- Animation tells the body what to do.
- VFX tells the player what changed.
- Sound tells the player what landed.

## Audit Checklist
- Does server own gameplay outcome?
- Are animation markers trusted only as feedback?
- Are VFX pooled or destroyed?
- Are particle counts bounded?
- Are sounds cleaned?
- Are tweens canceled or completed?
- Does mobile have a reduced path?
- Does the effect replicate too much data?
- Can the player understand hit, miss, reward, or failure?
- Are Animator / AnimationController choices correct?
- Are TweenService, particles, and sounds cleaned up?

## Output Rule
- For combat or ability systems, include animation/VFX flow.
- For generated code, separate gameplay authority from visual feedback.
- For audit, report unbounded VFX, leaked tracks, leaked sounds, and excessive replication.
