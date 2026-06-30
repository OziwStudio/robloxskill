# Template Catalog

## Purpose

- File ini membuktikan template genre siap pakai.
- Baca hanya saat `!template`, `!gamefull`, `!newgame`, atau genre game diminta.
- Jangan paste isi template penuh ke chat.
- Pakai template sebagai sumber arsitektur, sistem, data, UI, monetization, dan launch plan.

## Ready Templates

| Template | Path | Use |
|---|---|---|
| Scaffold | `Templates/game-scaffold.md` | struktur game lengkap |
| Simulator | `Templates/genre-simulator.md` | loop click, upgrade, pet, rebirth |
| Tycoon | `Templates/genre-tycoon.md` | plot, dropper, conveyor, cash, unlock |
| Obby | `Templates/genre-obby.md` | checkpoint, stage, timer, reward |
| RPG | `Templates/genre-rpg.md` | quest, level, combat, inventory |
| Horror | `Templates/genre-horror.md` | atmosphere, objective, chase, tension |
| Battle Royale | `Templates/genre-battle-royale.md` | lobby, match, zone, loot, elimination |

## Read Matrix

| Request | Read |
|---|---|
| New complete game | `Templates/game-scaffold.md` |
| Specific genre | `Templates/game-scaffold.md` + one genre template |
| Custom genre | `Templates/game-scaffold.md` + `EngineerDeepin/References/GameDesignLiveOps.md` |
| Monetized game | template + `EngineerDeepin/References/MonetizationAndEconomy.md` |
| Publish-ready game | template + `EngineerDeepin/References/TestingQA.md` + `EngineerDeepin/References/ProductionGates.md` |

## Extraction Rule

- Extract only the needed section.
- Keep generated output under current Roblox Skill Pluz architecture rules.
- Use current naming, header, debug, comment, and folder rules.
- Add missing assumptions briefly.
- If a template file is missing, name the missing path exactly.
