# Mesh Fidelity And LOD

## Purpose

- Use for visual audits, MeshPart, large maps, mobile performance, asset import, and world optimization.
- Focus on `RenderFidelity`, `CollisionFidelity`, and `LevelOfDetail`.
- Read with `references/Performance.md` when the bottleneck is rendering, physics, part count, mesh, or mobile.

## Priority

1. Physics safety.
2. Server performance.
3. Client memory.
4. Visual clarity.
5. Asset maintainability.

## CollisionFidelity

- `CollisionFidelity` controls the collision shape of a MeshPart.
- Do not use precise collision for decoration.
- Small decoration should use `CanCollide = false`.
- Large decorative objects should use `Box` or `Hull`.
- Use precise collision only for gameplay that truly needs accurate shape.
- Avoid `PreciseConvexDecomposition` on complex meshes frequently touched by players.
- Separate the visual mesh and invisible collision parts when collision shape needs control.

## RenderFidelity

- `RenderFidelity` controls MeshPart visual quality.
- High-poly meshes should use `Automatic` or `Performance` when far quality is not important.
- Use high quality only for assets near the camera or focal objects.
- Do not treat low-poly style as proof of lightweight performance.
- Measure triangle count, materials, texture size, transparency, and shadows.

## LevelOfDetail

- LOD reduces object detail based on distance, density, or device.
- For static assets, prioritize `RenderFidelity` and lightweight mesh design.
- For NPCs, pets, VFX, and dynamic decoration, create manual LOD.
- Manual LOD may use `Full`, `Simple`, `Billboard`, or `Hidden` modes.
- Mobile should use more aggressive LOD distances.
- Do not let LOD change gameplay outcomes.

## Suggested Defaults

| Asset | Collision | Render | LOD |
|---|---|---|---|
| Tree leaves | `CanCollide = false` | `Automatic` | simple far mesh |
| Wall decoration | `CanCollide = false` | `Performance` | hidden at far distance |
| Rock obstacle | `Hull` | `Automatic` | simple far mesh |
| Gameplay platform | simple parts or `Box` | `Automatic` | no gameplay-changing LOD |
| Pet/NPC cosmetic | `CanCollide = false` | `Performance` | billboard or hidden far |
| Hero object | controlled collision | `Precise` only if needed | keep readable |

## Audit Checklist

- Does this mesh need collision?
- Can collision be `Box`, `Hull`, or disabled?
- Is precise collision required by gameplay?
- Is this mesh high-poly?
- Can render fidelity be `Automatic` or `Performance`?
- Does mobile need simpler visual distance?
- Can distant dynamic objects become billboard or hidden?
- Does LOD preserve gameplay fairness?
- Are shadows, transparency, particles, and textures bounded?

## Output Rule

- For audits, report `CollisionFidelity`, `RenderFidelity`, and `LOD` separately.
- For generated maps, include mesh fidelity notes in file tree or setup notes.
- For code, keep LOD client-side unless gameplay authority is involved.
- For gameplay objects, server must not depend on client-only LOD.
