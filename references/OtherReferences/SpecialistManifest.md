# Specialist Manifest

## Purpose
- This file proves Deepin has separate specialists.
- Read only when the user asks for the specialist list or when auditing skill completeness.
- During normal work, read only the relevant specialist.

## Specialist Files
| Specialist | Path | Responsibility |
|---|---|---|
| Architecture | `references/Architecture.md` | boundaries, folders, ownership, scaling |
| Luau | `references/Luau.md` | strict/nonstrict, modules, typing, lifecycle |
| Networking | `references/Networking.md` | remotes, replication, prediction, rate limits |
| Security | `references/Security.md` | exploit paths, authority, validation, permissions |
| Data | `references/Data.md` | DataStore, ProfileStore, migration, receipts |
| Performance | `references/Performance.md` | memory, CPU, loops, streaming, profiling |
| Gameplay | `references/Gameplay.md` | combat, inventory, economy, progression |
| UIUX | `references/UIUX.md` | UI state, input, responsiveness, accessibility |
| Framework | `EngineerDeepin/Specialists/Framework.md` | Knit, ProfileStore, Fusion, Rojo only when enabled |
| Reviewer | `EngineerDeepin/Specialists/Reviewer.md` | final audit, regression, production readiness |

## Activation Rule
- Use `Deepin: SpecialistName` for one specialist.
- Use `!bugfix` for Luau + Reviewer + needed domain.
- Use `!vulnfix` for Security + Networking + Data when relevant.
- Use `!gamefull` for Architecture + Gameplay + Data + Networking + Security + Performance + UIUX + Reviewer.
- Do not load Framework unless framework support is enabled.
- Read `references/ImplementationPatterns.md` when specialist output needs direct code examples.

## Completeness Gate
- All 10 specialist files must exist.
- Each specialist file must have a clear scope.
- Each specialist must route to references only when needed.
- Deepin must not merge specialist detail into the main agent file.
- Code examples must live in references, not duplicated across every specialist file.
