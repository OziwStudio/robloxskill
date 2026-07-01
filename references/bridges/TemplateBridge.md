# Template Bridge

## Purpose
- This bridge connects Deepin to the bundled root `templates/` folder.
- Ready-to-use genre templates live in `templates/GameTemplates/`.
- Do not search templates in adoption folders.
- Do not recreate long templates when the template file already exists.
- Read a template only when the genre or scaffold needs detail.
- This file also acts as the template catalog.

## Expected Folder
```text
templates/
  GameTemplates/game-scaffold.md
  GameTemplates/genre-simulator.md
  GameTemplates/genre-tycoon.md
  GameTemplates/genre-obby.md
  GameTemplates/genre-rpg.md
  GameTemplates/genre-horror.md
```

## Command Router
| Command | Template |
|---|---|
| `!template scaffold` | `templates/GameTemplates/game-scaffold.md` |
| `!template simulator` | `templates/GameTemplates/genre-simulator.md` |
| `!template tycoon` | `templates/GameTemplates/genre-tycoon.md` |
| `!template obby` | `templates/GameTemplates/genre-obby.md` |
| `!template rpg` | `templates/GameTemplates/genre-rpg.md` |
| `!template horror` | `templates/GameTemplates/genre-horror.md` |

## Ready Check
- `templates/GameTemplates/game-scaffold.md` must exist.
- `templates/GameTemplates/genre-simulator.md` must exist.
- `templates/GameTemplates/genre-tycoon.md` must exist.
- `templates/GameTemplates/genre-obby.md` must exist.
- `templates/GameTemplates/genre-rpg.md` must exist.
- `templates/GameTemplates/genre-horror.md` must exist.

## Read Rule
- Read `templates/GameTemplates/game-scaffold.md` for new complete game.
- Read one genre template only.
- If user asks custom genre, use scaffold plus `GameDesign.md`.
- If template is missing, say which file is missing and continue with built-in `GameDesign.md` if acceptable.

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

## Catalog Rule
- Read only when `!template`, `!gamefull`, `!newgame`, or a game genre is requested.
- Do not paste full template content into chat.
- Use templates as the source for architecture, systems, data, UI, monetization, and launch plans.

## Ready Templates
| Template | Path | Use |
|---|---|---|
| Scaffold | `templates/GameTemplates/game-scaffold.md` | complete game structure |
| Simulator | `templates/GameTemplates/genre-simulator.md` | loop click, upgrade, pet, rebirth |
| Tycoon | `templates/GameTemplates/genre-tycoon.md` | plot, dropper, conveyor, cash, unlock |
| Obby | `templates/GameTemplates/genre-obby.md` | checkpoint, stage, timer, reward |
| RPG | `templates/GameTemplates/genre-rpg.md` | quest, level, combat, inventory |
| Horror | `templates/GameTemplates/genre-horror.md` | atmosphere, objective, chase, tension |

## Read Matrix
| Request | Read |
|---|---|
| New complete game | `templates/GameTemplates/game-scaffold.md` |
| Specific genre | `templates/GameTemplates/game-scaffold.md` + one genre template |
| Custom genre | `templates/GameTemplates/game-scaffold.md` + `GameDesign.md` |
| Monetized game | template + `references/topics/MonetizationAndEconomy.md` |
| Publish-ready game | template + `references/topics/TestingQA.md` + `references/Production.md` |

## Extraction Rule
- Extract only the needed section.
- Keep generated output under current Roblox Skill Pluz architecture rules.
- Use current naming, header, debug, comment, and folder rules.
- Add missing assumptions briefly.
- If a template file is missing, name the missing path exactly.
