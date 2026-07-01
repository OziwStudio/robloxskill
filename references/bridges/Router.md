# Adopted Router

## Purpose
- This router adopts strengths from other Roblox skills.
- Use this router only when Deepin is active.
- Choose the smallest sufficient specialist set.
- Deepin has 10 specialists adopted from Roblox specialist skills.
- Specialist files are read only when active.

## Engagement
| Mode | Use |
|---|---|
| `Express` | quick answer, small fix |
| `Standard` | normal system or explanation |
| `Thorough` | multi-file system, risky feature |
| `Meticulous` | audit, exploit, data, monetization |

## Missing Info
- `Critical`: blocks correctness or safety.
- `Degraded`: answer still useful with assumptions.
- `Optional`: does not change core solution.

## Specialist Routing
| Intent | Specialists |
|---|---|
| Build system | Architecture, Luau, Reviewer |
| Existing code migration | Architecture, Luau, Reviewer |
| Remote feature | Networking, Security, Luau |
| Combat | Gameplay, Networking, Security, Performance |
| Inventory | Gameplay, Data, Security, UIUX |
| Shop | Data, Security, Gameplay, UIUX |
| Trading | Data, Security, Networking, Reviewer |
| DataStore | Data, Security, Reviewer |
| Receipt | Data, Security, Reviewer |
| UI | UIUX, Performance, Gameplay |
| Performance issue | Performance, Networking, Luau |
| Security issue | Security, Networking, Data |
| Code review | Reviewer, relevant domain |
| Publish readiness | Reviewer, Security, Data, Performance |
| Modern Roblox API | Architecture, Luau, Networking, Performance, Reviewer |
| Open Cloud or LiveOps | Data, Security, Architecture, Reviewer |
| Modern UI API | UIUX, Performance, Reviewer |
| Avatar modernization | Gameplay, Networking, Security, Performance |
| Animation or VFX | Gameplay, Performance, UIUX, Reviewer |
| Mesh fidelity or LOD | Performance, Architecture, Reviewer |
| Community safety or policy | Security, UIUX, Reviewer |
| Complete game | Architecture, Gameplay, Data, Networking, Security, Performance, UIUX, Reviewer |
| Template-based game | Architecture, Gameplay, Reviewer |
| Testing and QA | Reviewer, Luau, relevant domain |
| Game design | Gameplay, UIUX, Reviewer |
| Studio automation | Architecture, Reviewer |

## Specialist File Map
| Specialist | Read |
|---|---|
| Luau | `references/Luau.md` |
| Architecture | `references/Architecture.md` |
| Networking | `references/Networking.md` |
| Security | `references/Security.md` |
| Performance | `references/Performance.md` |
| Data | `references/Data.md` |
| Gameplay | `references/Gameplay.md` |
| UIUX | `references/UIUX.md` |
| Framework | `references/Framework.md` only when enabled |
| Reviewer | `references/Reviewer.md` |

## Multi-Specialist Protocol
1. Determine objective.
2. Select relevant specialists.
3. Read only selected specialist files.
4. Combine their gates.
5. Resolve conflicts by priority.
6. Produce one unified answer.

## Conflict Priority
1. Security
2. Data Integrity
3. Server Authority
4. Correctness
5. Scalability
6. Performance
7. Maintainability
8. Developer Experience

## Brownfield Rule
- Detect existing architecture before changing it.
- Preserve user changes.
- Prefer local pattern over new abstraction.
- Migrate in small steps.
- Mention breaking change before applying it.

## MCP Awareness
- If Roblox MCP/tools exist, use them for project inspection.
- If MCP is missing, use filesystem and user-provided code.
- Do not make Deepin depend on unavailable tools.
- Read `references/StudioModes.md` for Studio automation or MCP requests.

## Template Awareness
- Read `references/bridges/TemplateBridge.md` when user asks `!template`, `!gamefull`, scaffold, or genre game.
- Read `TemplateBridge.md` to prove template availability and choose a genre template.
- Use bundled root `templates/` for template-based game generation.
- Do not require templates for normal answers.

## Specialist Completeness
- Read `references/bridges/SpecialistManifest.md` when user asks whether Deepin specialist files are complete.
- Do not claim specialist files are missing unless `references/*.md` is actually absent.

## Modern Roblox Rule
- Treat newest APIs as optional until verified.
- Prefer stable services for generated code.
- Add modern API path only when user requests or project already uses it.
- Read `references/topics/ModernRoblox.md` for modern API decisions.
- Provide fallback when API confidence is not `Stable`.
- For 2026 API claims, classify confidence and avoid hard requirements unless verified.

## Missed Documentation Rule
- Read `references/topics/AnimationVFX.md` for animation, VFX, SFX, particle, tween, camera shake, cutscene, ability feedback, and combat polish tasks.
- Read `references/topics/MeshFidelityLOD.md` for `RenderFidelity`, `CollisionFidelity`, `LevelOfDetail`, mesh-heavy maps, pets, NPCs, cosmetics, and mobile rendering.
- Read `references/topics/CommunitySafety2026.md` for youth audience, chat, voice, UGC, ads, subscriptions, reporting, or policy-sensitive systems.

## Self Check
- Does server own state?
- Does data save safely?
- Can remote be spammed?
- Can client forge value?
- Can receipt duplicate?
- Can connection leak?
- Can mobile performance survive?
- Can the system be extended?



