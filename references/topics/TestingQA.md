# Testing QA

## Purpose

- Use for test plans, bugfix regression, publish readiness, CI, and smoke testing.
- Prioritize testing for data, purchases, remotes, combat math, and core gameplay.
- Use for unit, integration, and manual verification on Roblox projects.
- Keep test guidance tied to the smallest verifiable path.

## Test Priority

1. Data save/load.
2. ProcessReceipt and purchases.
3. Remote validation.
4. Server authority.
5. Combat and reward math.
6. Inventory transactions.
7. UI critical flows.
8. Mobile controls.
9. Logging and error paths.
10. Shutdown and recovery.

## Test Types

| Type | Use |
|---|---|
| Unit | pure logic modules |
| Integration | multiple modules and data flow |
| Remote flow | client intent to server result |
| Smoke | game starts without errors |
| Manual | mobile, multiplayer, UX, monetization prompt |
| Regression | bug must not return |
| Acceptance | feature meets expected outcome |

## Coverage Focus
- Data integrity.
- Purchase safety.
- Remote safety.
- Damage math.
- Inventory mutation.
- Spawn and respawn flow.
- Mobile input.
- Error handling.
- Shutdown safety.

## TestEZ
- Use `.spec.luau` for test files.
- Place spec files alongside or mirroring the module they test.
- Use `describe`, `beforeEach`, `afterEach`, and `it`.
- Keep assertions focused and deterministic.
- Keep setup and teardown obvious.
- Keep each spec focused on one behavior.
- Use a root container that discovery can scan reliably.

```luau
return function()
	describe("CurrencyManager", function()
		local CurrencyManager

		beforeEach(function()
			CurrencyManager = require(script.Parent.CurrencyManager)
		end)

		it("should initialize zero gold", function()
			expect(CurrencyManager.newPlayerData().gold).to.equal(0)
		end)
	end)
end
```

## Assertion Patterns
- `expect(value).to.equal(expected)`
- `expect(value).to.be.ok()`
- `expect(value).to.be.a("table")`
- `expect(value).never.to.equal(unexpected)`
- `expect(function() ... end).to.throw()`
- `expect(value).to.be.near(value, tolerance)`

## Mock Rule
- Mock engine services instead of touching live services.
- Mock DataStore, MarketplaceService, Players, remotes, and signals.
- Keep mock behavior minimal and deterministic.
- Reset mock state between tests.

### Mock Shape
```luau
local MockDataStore = {}

function MockDataStore.new()
	return {
		GetAsync = function(_, key)
			return nil
		end,
		SetAsync = function(_, key, value)
			return true
		end,
	}
end
```

## Testable Code Rule

- Keep pure logic separate from engine glue.
- Inject Roblox services for testable modules.
- Mock DataStore, MarketplaceService, Players, remotes, and signals.
- Use deterministic inputs.
- Reset state before each test.
- Avoid direct engine access inside pure functions.
- Keep state setup explicit in tests.
- Prefer data-in/data-out functions where possible.
- Move engine calls to module boundaries.

## Unit Tests
- Test pure modules first.
- Prefer math, config, transform, and validation code.
- Keep tests small and independent.
- Use unit tests for fast feedback.

## Integration Tests
- Test module interaction and data flow.
- Include remotes, persistence, and gameplay handoffs.
- Verify the full path, not only a helper.
- Use integration tests for the real glue code.
- Include failure paths, not only happy paths.

## Dependency Injection
- Pass services into constructors when a module needs engine access.
- Use `.init()` injection for singleton-style modules.
- Prefer mocks in tests instead of live services.
- Inject `Players`, `DataStoreService`, `MarketplaceService`, and similar services at the boundary.

### Constructor Injection
```luau
local Service = {}
Service.__index = Service

function Service.new(dataStoreService)
	local self = setmetatable({}, Service)
	self._store = dataStoreService:GetDataStore("X")
	return self
end
```

## Test Runner
- Keep a runner script in Studio when needed.
- Point TestEZ at the correct root container.
- Keep test discovery predictable.
- Keep runner code in a known server location.
- Make runner output easy to read.

### Studio Runner
```luau
local TestEZ = require(game.ReplicatedStorage.DevPackages.TestEZ)
local results = TestEZ.TestBootstrap:run({
	game.ReplicatedStorage.Shared,
	game.ServerScriptService.Server,
})
```

## Bugfix Rule

1. Reproduce bug.
2. Add failing test or manual repro.
3. Fix smallest area.
4. Re-run test.
5. Add regression note.
6. Keep the fix isolated.
7. Confirm no related path regressed.
8. Add the regression test before shipping.
9. Note residual risk when the fix is incomplete.

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
- No remote spam.
- No duplicate grants.
- No save corruption after leave.
- No broken respawn flow.
- No stale UI after server update.
- No broken retry behavior.
- No missing mobile fallback.

## CI Rule

- If Project Management support is enabled, use Rojo/Wally/Aftman/Selene/StyLua/TestEZ/Lune only when project uses them.
- Do not force CI on Studio-only users.
- Suggest CI for production teams.
- Use linting and formatting before test runs when available.
- Keep automation optional for small projects.
- Use CI to prevent regressions on production teams.
- Keep the pipeline simple enough to maintain.
- Use tests before publish when possible.

## Smoke Rule
- Confirm the game starts cleanly.
- Confirm the first player can load.
- Confirm the core loop can complete once.
- Confirm no errors appear in output.
- Confirm no infinite yield warnings appear.

## Test Data Rule
- Use deterministic fixtures.
- Reset shared state between tests.
- Avoid hidden dependencies between specs.
- Prefer simple mock services.

## Regression Rule
- Turn every real bug into a test or checklist item.
- Keep regression coverage near the affected module.
- Re-run the critical path after every fix.
- Keep regression notes close to the code that changed.

## Failure Path Rule
- Test failure as a first-class case.
- Validate grant failure.
- Validate save failure.
- Validate disconnected player behavior.
- Validate missing remote payloads.
- Validate duplicate receipt handling.

## Debug Workflow

- Collect the raw error output first.
- Capture exact error text, stack trace, script path, and line number.
- Reproduce the bug before changing code when possible.
- Read surrounding source and related callers or modules.
- Classify the bug before fixing it.
- Keep fixes minimal and focused.
- Re-run the failing path after each change.
- Stop after a bounded number of failed attempts and write a findings summary.

## Debug Iteration Rule

- Limit repeated automatic debug passes.
- Track hypothesis, attempted fix, and result for each pass.
- Escalate when the same issue persists after multiple attempts.
- Keep regression notes close to the changed code.

## Bug Report Rule

- Record exact error, script path, and line.
- Record root cause category.
- Record the fix applied.
- Record iterations required.
- Record preventive advice for the same class of bug.

## Output Format

```text
[test_scope]
[critical_paths]
[automated_tests]
[manual_tests]
[regression_checks]
[remaining_risk]
```
