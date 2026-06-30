# Complete Game Layer

## Purpose

- This reference makes Deepin complete for full Roblox game creation.
- Use when the user asks for a new game, complete system, game design, genre, publish, live-service, or broad audit.
- Load this file before choosing a long technical workflow.
- Still use other files only when needed.

## Complete Game Stack

1. Core loop.
2. FTUE.
3. Progression.
4. Meta-progression.
5. Social surface.
6. Data persistence.
7. Server authority.
8. Networking.
9. UI/UX.
10. Monetization.
11. Performance.
12. Testing.
13. Publish readiness.
14. Update strategy.

## Intent Router

| Intent | Read |
|---|---|
| New game | `WorkflowPlaybooks.md`, `GameDesignLiveOps.md`, `TemplateBridge.md`, `TemplateCatalog.md`, `GenrePlaybooks.md` |
| Simulator | `TemplateBridge.md`, `TemplateCatalog.md`, `GenrePlaybooks.md`, `Gameplay.md`, `Data.md` |
| Tycoon | `TemplateBridge.md`, `TemplateCatalog.md`, `GenrePlaybooks.md`, `Performance.md`, `Data.md` |
| Obby | `TemplateBridge.md`, `TemplateCatalog.md`, `GenrePlaybooks.md`, `Gameplay.md`, `UIUX.md` |
| RPG | `TemplateBridge.md`, `TemplateCatalog.md`, `GenrePlaybooks.md`, `Gameplay.md`, `Data.md`, `Networking.md` |
| Horror | `TemplateBridge.md`, `TemplateCatalog.md`, `GenrePlaybooks.md`, `GameDesignLiveOps.md`, `Performance.md` |
| Battle royale | `TemplateBridge.md`, `TemplateCatalog.md`, `GenrePlaybooks.md`, `Networking.md`, `Security.md`, `Performance.md` |
| Monetization | `MonetizationAndEconomy.md`, `ProductionGates.md`, `Data.md`, `Security.md` |
| Testing | `TestingQA.md`, `WorkflowPlaybooks.md` |
| Studio automation | `StudioModes.md` |
| Publish | `WorkflowPlaybooks.md`, `ProductionGates.md`, `TestingQA.md`, `GameDesignLiveOps.md` |

## New Game Order

1. Detect genre.
2. Load template bridge and template catalog.
3. Define scope: MVP, standard, full, live-service.
4. Define file tree.
5. Generate scaffold.
6. Generate core systems.
7. Add data and remotes.
8. Add UI.
9. Add monetization only after fairness.
10. Add tests and playtest checklist.
11. Run production gates.

## MVP Rule

- Make first playable fast.
- Keep server authority from day one.
- Keep data safe from day one.
- Keep monetization optional.
- Add complexity after loop works.

## Complete Game Gate

- Core loop works in 10 seconds.
- First reward happens within 30 seconds.
- Player knows next action.
- Mobile is playable.
- Social surface exists.
- Data cannot be lost by normal leave/rejoin.
- Remotes cannot grant value directly.
- Purchases are server-granted.
- Publish checklist passes.
