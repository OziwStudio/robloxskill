# Template Bridge

## Purpose

- This bridge connects Deepin to the bundled root `Templates/` folder.
- Ready-to-use genre templates must live in root `Templates/`.
- Do not search templates in adoption folders.
- Do not recreate long templates when the template file already exists.
- Read a template only when the genre or scaffold needs detail.
- For the complete template list, read `references/TemplateCatalog.md`.

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
| `!template scaffold` | `Templates/GameTemplates/game-scaffold.md` |
| `!template simulator` | `Templates/GameTemplates/genre-simulator.md` |
| `!template tycoon` | `Templates/GameTemplates/genre-tycoon.md` |
| `!template obby` | `Templates/GameTemplates/genre-obby.md` |
| `!template rpg` | `Templates/GameTemplates/genre-rpg.md` |
| `!template horror` | `Templates/GameTemplates/genre-horror.md` |
| `!template battleroyale` | `Templates/GameTemplates/genre-battle-royale.md` |

## Ready Check

- `Templates/GameTemplates/game-scaffold.md` must exist.
- `Templates/GameTemplates/genre-simulator.md` must exist.
- `Templates/GameTemplates/genre-tycoon.md` must exist.
- `Templates/GameTemplates/genre-obby.md` must exist.
- `Templates/GameTemplates/genre-rpg.md` must exist.
- `Templates/GameTemplates/genre-horror.md` must exist.
- `Templates/GameTemplates/genre-battle-royale.md` must exist.

## Read Rule

- Read `Templates/GameTemplates/game-scaffold.md` for new complete game.
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

# Template Catalog

## Purpose

- This file proves ready-to-use genre templates exist.
- Read only when `!template`, `!gamefull`, `!newgame`, or a game genre is requested.
- Do not paste full template content into chat.
- Use templates as the source for architecture, systems, data, UI, monetization, and launch plans.

## Ready Templates

| Template | Path | Use |
|---|---|---|
| Scaffold | `Templates/GameTemplates/game-scaffold.md` | complete game structure |
| Simulator | `Templates/GameTemplates/genre-simulator.md` | loop click, upgrade, pet, rebirth |
| Tycoon | `Templates/GameTemplates/genre-tycoon.md` | plot, dropper, conveyor, cash, unlock |
| Obby | `Templates/GameTemplates/genre-obby.md` | checkpoint, stage, timer, reward |
| RPG | `Templates/GameTemplates/genre-rpg.md` | quest, level, combat, inventory |
| Horror | `Templates/GameTemplates/genre-horror.md` | atmosphere, objective, chase, tension |
| Battle Royale | `Templates/GameTemplates/genre-battle-royale.md` | lobby, match, zone, loot, elimination |

## Read Matrix

| Request | Read |
|---|---|
| New complete game | `Templates/GameTemplates/game-scaffold.md` |
| Specific genre | `Templates/GameTemplates/game-scaffold.md` + one genre template |
| Custom genre | `Templates/GameTemplates/game-scaffold.md` + `references/GameDesignLiveOps.md` |
| Monetized game | template + `references/MonetizationAndEconomy.md` |
| Publish-ready game | template + `references/TestingQA.md` + `references/ProductionGates.md` |

## Extraction Rule

- Extract only the needed section.
- Keep generated output under current Roblox Skill Pluz architecture rules.
- Use current naming, header, debug, comment, and folder rules.
- Add missing assumptions briefly.
- If a template file is missing, name the missing path exactly.





