# SoundService
> Roblox Luau reference for audio orchestration, pooling, routing, and lifecycle control.
> Use this file as the single SoundService guide for AI reasoning.

## 1. Purpose

SoundService is a centralized audio orchestration layer for Roblox.
It helps manage music, SFX, UI audio, ambience, and spatial playback without scattering unmanaged `Sound` objects everywhere.

Use it when you want:
- grouped audio mixing
- sound pooling
- crossfading
- centralized sound IDs
- cleanup of orphaned audio

Do not use it as a replacement for the engine `Sound` instance.
It is a control layer, not the audio itself.

## 2. Core Principles

1. Route audio through groups.
2. Reuse sound instances.
3. Keep audio logic server/client aware.
4. Every sound needs a clear end state.
5. Sound IDs should be data-driven.
6. Separate 2D and 3D audio.
7. Audio failure should never stop gameplay.
8. Cleanup must be explicit.

## 3. API Surface

### 3.1 Play

`Play(soundName: string, config: table)`

- plays a sound from the registry
- may override volume, parent, loop state, or position

### 3.2 Stop

`Stop(soundName: string, fadeTime: number?)`

- stops one or more active sounds
- supports optional fade-out

### 3.3 SetGroupVolume

`SetGroupVolume(groupName: string, volume: number)`

- changes the mix level for a sound group
- clamp values to a safe range

### 3.4 Preload

`Preload(soundNames: {string})`

- batches asset preloading
- use before audio-heavy gameplay starts

## 4. Sound Group Hierarchy

Use a stable group layout.

- Master
- Music
- SFX
- UI
- Combat
- Ambient

Rules:
- every managed sound should belong to a group
- never mix volume directly per sound when a group will do
- keep group names stable across the project

## 5. Lifecycle Pattern

The correct flow is:
1. initialize groups
2. create or warm the pool
3. acquire a sound
4. set properties
5. play
6. on end, reset and return to pool

Rules:
- do not create disposable sounds repeatedly
- do not leave ended sounds parented
- do not keep stale playback state

## 6. Pool Pattern

Pooling is the default.

Rules:
- reuse active sound instances
- keep a cap on total live sounds
- recycle the oldest sound when the pool is full
- reset loop, volume, parent, and sound id before reuse

Pooling reduces memory churn and avoids noisy instance creation.

## 7. Replication Rules

| Action | Server | Client |
|---|---|---|
| request play | signal intent | execute playback |
| mix control | shared config | apply to local groups |
| sound creation | avoid for client feedback | use local pool |

Rules:
- server should signal intent when the audio is gameplay-relevant
- client should execute cosmetic or feedback audio
- do not rely on client audio state for authority

## 8. Component Pattern

For spatial or attached audio, use a component-like pattern.

Rules:
- attach sound logic to the owning instance
- detach on destroy or stream out
- clean up hanging sounds immediately
- handle streaming safely

## 9. Manager Pattern

Use one audio manager for production audio flow.

Responsibilities:
- preload queue
- pool ownership
- crossfade logic
- active sound tracking
- global stop behavior

The manager should know when to play and stop, not why the game exists.

## 10. Crossfade and Fading

Do not hard stop music when a smoother transition is needed.

Rules:
- fade old sound out
- reset old sound after stop
- start new sound at low volume
- fade new sound up

Crossfading is the right default for music changes.

## 11. Performance Rules

- use pooling
- cap active sounds
- keep long music segmented when possible
- disconnect or recycle end handlers
- avoid more than a few sound starts in one frame
- preload important assets early

## 12. Security Rules

- never trust the client to prove game-state events through audio
- audio is feedback, not authority
- sanitize sound IDs before use
- keep sound registry values centralized

## 13. Sound Registry

Keep all sound IDs in one module.

Rules:
- no raw ID littering
- no hardcoded asset strings across gameplay code
- reference the registry everywhere

## 14. Common Failures

### 14.1 Audio leak

Problem: new sound instance every click.
Fix: use the pool.

### 14.2 Loop stuck on

Problem: recycled sound keeps looping.
Fix: reset all playback properties.

### 14.3 Hardcoded mix

Problem: volume is edited per sound instead of per group.
Fix: use sound groups.

### 14.4 Loading delay

Problem: audio plays late.
Fix: preload assets.

## 15. AI Behavior Rules

- prioritize pooling over fresh instantiation
- ask for a group target when audio is being designed
- suggest crossfade for background music
- separate registry data from playback logic
- warn about audio churn and leaked connections

## 16. Mental Model

Think in three layers:
- registry layer: what sounds exist
- pool layer: what instances are available
- mixer layer: how volume is controlled

## 17. Minimal Checklist

- sound IDs are centralized
- groups are assigned
- pool exists
- fades are used for music transitions
- orphaned sounds are cleaned
- streaming audio is handled safely
- master and submix groups work

## 18. Final Rule

SoundService should make audio feel polished and controlled.
If you are manually parenting fresh `Sound` instances everywhere, the architecture is too loose.

