# Performance

## Purpose

- Use for CPU, memory, rendering, UI, VFX, networking, mobile, and scale audits.

## Priority

1. Measure or infer bottleneck class.
2. Fix leaks.
3. Reduce unbounded work.
4. Reduce replication.
5. Reduce rendering load.
6. Keep code maintainable.

## Server

- Avoid excessive Heartbeat.
- Avoid polling when events work.
- Bound table growth.
- Clean per-player state.
- Use task scheduling carefully.

## Client

- Mobile is first-class.
- Avoid hardcoded UI.
- Reduce visible part count.
- Use StreamingEnabled for large worlds.
- Degrade VFX on low-end devices.
- For mesh-heavy maps, read `MeshFidelityLOD.md`.

## Mesh And LOD

- Audit `CollisionFidelity` on every MeshPart that can collide.
- Use `Box`, `Hull`, or `CanCollide = false` for decoration.
- Avoid precise mesh collision unless gameplay needs it.
- Audit `RenderFidelity` for high-poly assets.
- Prefer `Automatic` or `Performance` for non-hero mesh visuals.
- Use manual LOD for pets, NPCs, cosmetics, and distant dynamic objects.
- LOD must not change server-owned gameplay outcomes.

## VFX

- For detailed animation/VFX rules, read `AnimationVFX.md`.
- Keep emitters bounded.
- Keep particle texture small.
- Pool frequent VFX.
- Destroy one-shot sounds.
- Disconnect camera shake.
- Restore camera after cutscene.
- Cancel or complete tweens before destroying instances.
- Reduce effects on low-end devices.

## Network

- Reduce remote frequency.
- Reduce payload size.
- Avoid replication of unnecessary state.
- Use interest and streaming where suitable.

## Anti-Patterns

- Infinite loop.
- Remote spam.
- Connection leak.
- Massive part count.
- Unbounded VFX.
- Thousands of tweens.
- PointLight spam with shadows.
- Precise collision on decorative mesh.
- High-poly mesh with no LOD plan.
