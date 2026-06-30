# Animation And VFX

## Purpose

- Use for animation, VFX, SFX timing, particles, tweens, camera shake, cutscenes, abilities, combat feedback, and visual polish.
- Read with `Performance.md` when effects may affect FPS, memory, mobile, or network replication.
- Read with `Gameplay.md` when VFX represents gameplay feedback.
- Read with `Security.md` when client VFX could be confused with server-owned outcomes.

## Priority

1. Gameplay clarity.
2. Performance safety.
3. Cleanup.
4. Network economy.
5. Visual polish.

## Animation Rules

- Server owns gameplay result.
- Client may play local animation feedback.
- Do not let animation events grant damage, reward, currency, or inventory.
- Use animation markers for timing feedback only unless server validates the outcome.
- Keep combat hit confirmation server-side.
- Stop or replace animation tracks when state changes.
- Clean character animation connections on respawn.
- Keep legacy Humanoid fallback unless modern avatar controller is confirmed.

## VFX Rules

- VFX is presentation, not authority.
- Spawn frequent VFX locally when possible.
- Replicate only the minimum event needed to show VFX.
- Pool repeated effects.
- Destroy one-shot emitters, beams, trails, sounds, and attachments.
- Bound particles by lifetime, rate, size, and texture cost.
- Disable or simplify heavy VFX on low-end devices.
- Keep mobile and crowded servers first-class.

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

## Output Rule

- For combat or ability systems, include animation/VFX flow.
- For generated code, separate gameplay authority from visual feedback.
- For audit, report unbounded VFX, leaked tracks, leaked sounds, and excessive replication.
