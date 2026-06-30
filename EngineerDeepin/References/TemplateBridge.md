# Template Bridge

## Purpose

- Bridge ini menghubungkan Deepin ke folder root `Templates/` yang sudah dibundel.
- Template genre siap pakai wajib berada di root `Templates/`.
- Jangan mencari template di folder adopsi.
- Jangan membuat ulang template panjang jika file template sudah ada.
- Baca template hanya saat genre atau scaffold membutuhkan detail.
- Untuk daftar template lengkap, baca `EngineerDeepin/References/TemplateCatalog.md`.

## Expected Folder

```text
Templates/
  game-scaffold.md
  genre-simulator.md
  genre-tycoon.md
  genre-obby.md
  genre-rpg.md
  genre-horror.md
  genre-battle-royale.md
```

## Command Router

| Command | Template |
|---|---|
| `!template scaffold` | `Templates/game-scaffold.md` |
| `!template simulator` | `Templates/genre-simulator.md` |
| `!template tycoon` | `Templates/genre-tycoon.md` |
| `!template obby` | `Templates/genre-obby.md` |
| `!template rpg` | `Templates/genre-rpg.md` |
| `!template horror` | `Templates/genre-horror.md` |
| `!template battleroyale` | `Templates/genre-battle-royale.md` |

## Ready Check

- `Templates/game-scaffold.md` must exist.
- `Templates/genre-simulator.md` must exist.
- `Templates/genre-tycoon.md` must exist.
- `Templates/genre-obby.md` must exist.
- `Templates/genre-rpg.md` must exist.
- `Templates/genre-horror.md` must exist.
- `Templates/genre-battle-royale.md` must exist.

## Read Rule

- Read `Templates/game-scaffold.md` for new complete game.
- Read one genre template only.
- If user asks custom genre, use scaffold plus `GameDesignLiveOps.md`.
- If template is missing, say which file is missing and continue with built-in `GenrePlaybooks.md` if acceptable.

## Extract From Template

- Core loop.
- Folder structure.
- Core systems.
- Data schema.
- Remotes.
- UI flow.
- Monetization strategy.
- Performance concerns.
- Launch checklist.

## Output Rule

- Do not paste full template back to user.
- Use bundled template as source for generated architecture.
- Preserve Roblox Skill Pluz header, footer, comment, debug, and folder rules.
- Adapt template paths to current architecture rules.
