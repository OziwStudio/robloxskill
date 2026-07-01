# Testing QA

## Purpose

- Use for test plans, bugfix regression, publish readiness, CI, and smoke testing.
- Prioritize testing for data, purchases, remotes, combat math, and core gameplay.

## Test Priority

1. Data save/load.
2. ProcessReceipt and purchases.
3. Remote validation.
4. Server authority.
5. Combat and reward math.
6. Inventory transactions.
7. UI critical flows.
8. Mobile controls.

## Test Types

| Type | Use |
|---|---|
| Unit | pure logic modules |
| Integration | multiple modules and data flow |
| Remote flow | client intent to server result |
| Smoke | game starts without errors |
| Manual | mobile, multiplayer, UX, monetization prompt |
| Regression | bug must not return |

## Testable Code Rule

- Keep pure logic separate from engine glue.
- Inject Roblox services for testable modules.
- Mock DataStore, MarketplaceService, Players, remotes, and signals.
- Use deterministic inputs.
- Reset state before each test.

## Bugfix Rule

1. Reproduce bug.
2. Add failing test or manual repro.
3. Fix smallest area.
4. Re-run test.
5. Add regression note.

## Manual QA Checklist

- Mobile UI fits.
- Touch controls work.
- 2+ players replicate state.
- Data loads for new player.
- Leave and rejoin keeps data.
- Rapid action spam is blocked.
- Inventory full case works.
- Purchase prompt, grant, duplicate receipt, and failure path work.
- No Output errors.
- No infinite yield warnings.

## CI Rule

- If Project Management support is enabled, use Rojo/Wally/Aftman/Selene/StyLua/TestEZ/Lune only when project uses them.
- Do not force CI on Studio-only users.
- Suggest CI for production teams.

## Output Format

```text
[test_scope]
[critical_paths]
[automated_tests]
[manual_tests]
[regression_checks]
[remaining_risk]
```
