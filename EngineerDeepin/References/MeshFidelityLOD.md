# Mesh Fidelity And LOD

## Purpose

- Pakai untuk audit visual, MeshPart, map besar, mobile performance, asset import, dan world optimization.
- Fokus pada `RenderFidelity`, `CollisionFidelity`, dan `LevelOfDetail`.
- Baca bersama `Performance.md` saat bottleneck ada di rendering, physics, part count, mesh, atau mobile.

## Priority

1. Physics safety.
2. Server performance.
3. Client memory.
4. Visual clarity.
5. Asset maintainability.

## CollisionFidelity

- `CollisionFidelity` mengontrol bentuk collision MeshPart.
- Jangan pakai collision presisi untuk dekorasi.
- Dekorasi kecil pakai `CanCollide = false`.
- Objek dekoratif besar pakai `Box` atau `Hull`.
- Pakai collision presisi hanya untuk gameplay yang benar-benar butuh bentuk akurat.
- Hindari `PreciseConvexDecomposition` pada mesh rumit yang sering disentuh pemain.
- Pisahkan visual mesh dan invisible collision part jika bentuk collision butuh kontrol.

## RenderFidelity

- `RenderFidelity` mengontrol kualitas visual MeshPart.
- Mesh high-poly pakai `Automatic` atau `Performance` bila kualitas jauh tidak penting.
- Pakai kualitas tinggi hanya untuk asset dekat kamera atau focal object.
- Jangan mengandalkan gaya low-poly sebagai bukti performa ringan.
- Ukur jumlah triangle, material, texture size, transparency, dan shadow.

## LevelOfDetail

- LOD mengurangi detail objek berdasarkan jarak, kepadatan, atau device.
- Untuk asset statis, utamakan `RenderFidelity` dan desain mesh yang ringan.
- Untuk NPC, pet, VFX, dan dekor dinamis, buat LOD manual.
- LOD manual boleh memakai mode `Full`, `Simple`, `Billboard`, `Hidden`.
- Mobile harus punya jarak LOD lebih agresif.
- Jangan membuat LOD mengubah gameplay outcome.

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
