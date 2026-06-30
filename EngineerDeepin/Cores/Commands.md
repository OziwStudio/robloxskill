# Commands

## Core

| Command | Action |
|---|---|
| `!deepin` | Activate Deepin and read `EngineerDeepin/DeepinEngineerAgent.md`. |
| `!Deepin` | Alias for `!deepin`. |
| `!read-deepin` | Read Deepin core only. |
| `!read-deepin-router` | Read `References/AdoptedRouter.md`. |
| `!read-workflow` | Read `References/WorkflowPlaybooks.md`. |
| `!read-gates` | Read `References/ProductionGates.md`. |
| `!read-complete-game` | Read `References/CompleteGameLayer.md`. |
| `!read-studio-modes` | Read `References/StudioModes.md`. |
| `!read-template-bridge` | Read `References/TemplateBridge.md`. |
| `!read-template-catalog` | Read `References/TemplateCatalog.md`. |
| `!read-testing` | Read `References/TestingQA.md`. |
| `!read-design` | Read `References/GameDesignLiveOps.md`. |
| `!read-monetization` | Read `References/MonetizationAndEconomy.md`. |
| `!read-genre` | Read `References/GenrePlaybooks.md`. |
| `!read-modern` | Read `References/ModernRoblox.md`. |
| `!read-animation-vfx` | Read `References/AnimationVFX.md`. |
| `!read-specialists` | Read `References/SpecialistManifest.md`. |
| `!read-patterns` | Read `References/ImplementationPatterns.md`. |
| `!read-mesh-lod` | Read `References/MeshFidelityLOD.md`. |
| `!read-safety2026` | Read `References/CommunitySafety2026.md`. |

## Engineer

| Command | Action |
|---|---|
| `Engineer: Basic` | Use default BasicEngineer. |
| `Engineer: Deepin` | Use DeepinEngineer. |
| `Deepin: Solo` | Use Deepin without specialist split. |
| `Deepin: Architecture` | Focus architecture. |
| `Deepin: Luau` | Focus Luau code quality. |
| `Deepin: Networking` | Focus remotes and replication. |
| `Deepin: Security` | Focus anti-exploit and authority. |
| `Deepin: Data` | Focus persistence and transactions. |
| `Deepin: Performance` | Focus `references/Performance.md`. |
| `Deepin: Gameplay` | Focus game systems. |
| `Deepin: UIUX` | Focus UI and client UX. |
| `Deepin: Reviewer` | Focus review and production readiness. |
| `Deepin: Framework` | Focus framework only when support is enabled. |

## Specialist Files

| Command | Read |
|---|---|
| `Deepin: Luau` | `EngineerDeepin/Specialists/Luau.md` |
| `Deepin: Architecture` | `EngineerDeepin/Specialists/Architecture.md` |
| `Deepin: Networking` | `EngineerDeepin/Specialists/Networking.md` |
| `Deepin: Security` | `EngineerDeepin/Specialists/Security.md` |
| `Deepin: Performance` | `references/Performance.md` |
| `Deepin: Data` | `EngineerDeepin/Specialists/Data.md` |
| `Deepin: Gameplay` | `EngineerDeepin/Specialists/Gameplay.md` |
| `Deepin: UIUX` | `EngineerDeepin/Specialists/UIUX.md` |
| `Deepin: Framework` | `EngineerDeepin/Specialists/Framework.md` only when enabled |
| `Deepin: Reviewer` | `EngineerDeepin/Specialists/Reviewer.md` |

## Task

| Command | Action |
|---|---|
| `!genfull` | Generate full system with file tree first. |
| `!gensnip` | Generate small snippet only. |
| `!bugfix` | Focus bug path, root cause, patch, regression. |
| `!vulnfix` | Focus exploit path, authority, validation, abuse patch. |
| `!audit` | Review architecture, security, data, performance, maintainability. |
| `!review` | Code review findings first. |
| `!perfcheck` | Performance audit. |
| `!publishcheck` | Publish readiness checklist. |
| `!monetcheck` | Monetization and receipt safety audit. |
| `!newgame` | New game planning workflow. |
| `!modernapi` | Check modern Roblox API path, confidence, and fallback. |
| `!gamefull` | Build complete game plan with design, scaffold, systems, tests, and launch gates. |
| `!template` | Use `TemplateBridge`, `TemplateCatalog`, and one matching template only. |
| `!testplan` | Create QA, regression, smoke, and manual testing plan. |
| `!sharpedges` | Check the 12 common Roblox production footguns. |
| `!mcp-mode` | Detect Full, Standard, or Offline Studio mode. |
| `!ftue` | Review first-time user experience and first 60 seconds. |
| `!retention` | Review return loops, social hooks, and LiveOps. |

## Reading

| Command | Action |
|---|---|
| `!read-architecture` | Read architecture reference. |
| `!read-networking` | Read networking reference. |
| `!read-security` | Read security reference. |
| `!read-data` | Read data reference. |
| `!read-performance` | Read performance reference. |
| `!read-vfx` | Read animation and VFX reference. |
| `!read-meshfidelity` | Read mesh fidelity and LOD reference. |
| `!read-community-safety` | Read community safety 2026 reference. |
| `!read-gameplay` | Read gameplay reference. |
| `!read-modernapi` | Read modern Roblox reference. |
| `!read-qa` | Read testing QA reference. |
| `!read-liveops` | Read game design LiveOps reference. |
| `!read-uiux` | Read UIUX reference. |
| `!read-luau` | Read Luau patterns reference. |
| `!read-systems` | Read system blueprint reference. |
| `!read-implementation-patterns` | Read concrete Deepin code patterns. |

## Support

| Command | Action |
|---|---|
| `!support-framework` | Enable Framework support. |
| `!support-management-project` | Enable Project Management support. |

## Generate Questions

- Ask: `Do you already have a reference system/code? Attach it if available.`
- Ask: `Are you using a framework, project management, or similar support?`
- Ask: `What complexity depth do you want: Shallow, Medium, or Deep?`
- Ask: `Generation depth: separated or all-in?`
- Ask: `Generation type: package or copy-paste?`

## Keyword Rules

- `deepin`: activate Deepin.
- `bugfix`: read workflow and focus root cause.
- `vulnfix`: read security and gates.
- `publish`: read workflow and gates.
- `monetization`: read workflow, data, security.
- `gamefull`: read CompleteGameLayer, TemplateBridge, TemplateCatalog, GameDesignLiveOps, TestingQA.
- `template`: read TemplateBridge, TemplateCatalog, and one matching `Templates/` file if present.
- `testing`: read TestingQA.
- `sharpedges`: read ProductionGates.
- `mcp`: read StudioModes.
- `ftue`: read GameDesignLiveOps.
- `retention`: read GameDesignLiveOps.
- `modernapi`: read ModernRoblox and verify API confidence.
- `opencloud`: read ModernRoblox, data, security.
- `liveops`: read ModernRoblox and data.
- `streaming`: read ModernRoblox and performance.
- `animation`: read AnimationVFX and `references/Performance.md`.
- `vfx`: read AnimationVFX and `references/Performance.md`.
- `sfx`: read AnimationVFX and `references/Performance.md`.
- `particles`: read AnimationVFX and `references/Performance.md`.
- `camera-shake`: read AnimationVFX and UIUX.
- `cutscene`: read AnimationVFX and UIUX.
- `renderfidelity`: read MeshFidelityLOD and `references/Performance.md`.
- `collisionfidelity`: read MeshFidelityLOD and `references/Performance.md`.
- `levelofdetail`: read MeshFidelityLOD and `references/Performance.md`.
- `lod`: read MeshFidelityLOD and `references/Performance.md`.
- `community-safety`: read CommunitySafety2026.
- `policy`: read CommunitySafety2026.
- `implementation`: read ImplementationPatterns.
- `code-example`: read ImplementationPatterns.
- `rate-limiter`: read ImplementationPatterns and Networking.
- `payload-validation`: read ImplementationPatterns and Security.
- `remote-handler`: read ImplementationPatterns and Networking.
- `schema`: read ImplementationPatterns and Data.
- `receipt`: read ImplementationPatterns, Data, Security.
- `server-authority`: read ImplementationPatterns, Gameplay, Security.
