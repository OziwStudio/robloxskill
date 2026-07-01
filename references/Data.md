# Data Specialist

## Scope

- DataStore.
- ProfileStore.
- Session locking.
- Save integrity.
- Migration.
- Recovery.
- Economy persistence.
- Receipt safety.

## Activate When

- User asks data, save, load, profile, inventory, economy, purchase, receipt, leaderboard, migration, backup, rollback, or duplication.

## Standards

- Read `references/ImplementationPatterns.md` when schema, migration, or receipt examples are needed.
- Use `Profile Schema` and `Receipt Idempotency` patterns.
- Player data is sacred.
- Favor safety over convenience.
- Use session locking for valuable data.
- Separate persistent and session state.
- Reconcile defaults.
- Version schemas.
- Migrate safely.
- Save with retry.
- Handle load failure.
- Keep receipt grants idempotent.
- Use Open Cloud only for external integration or LiveOps needs.
- Keep local config fallback for live config.
- Save before teleport when server hopping.
- Use separate test DataStore prefix in Studio.

## Review Gate

- Who owns data?
- Who mutates data?
- What happens on crash?
- What happens on throttle?
- What happens on old schema?
- Can duplicate grant happen?
- Can newer data be overwritten?
- Are Open Cloud scopes safe?
- Can teleport cause stale save?

## Anti-Patterns

- Missing session lock.
- Unsafe save.
- Save spam.
- Versionless profile.
- Massive profile.
- Duplicate save path.
- Race condition.
- Missing recovery.

## Output Focus

- For persistence: define schema and lifecycle.
- For receipt: define idempotency.
- For audit: lead with data-loss risk.
- Include schema version, reconciliation, save lifecycle, and failure behavior.

## Practical Principles

- Data changes should pass through one server-owned data service.
- Every profile field needs a default and migration story.
- Save frequency should be dirty-state based, not every event.
- Receipts need an idempotency key before granting value.
- Never store Instances, Vector3 userdata, or non-serializable objects directly.

## Schema Contract Example

```luau
export type PlayerData = {
	DataVersion: number,
	Coins: number,
	EquippedItemId: string?,
	Inventory: { [string]: number },
	Receipts: { [string]: boolean },
}
```

## Mutation Rule Example

```luau
local function addCoins(data: PlayerData, amount: number): boolean
	if amount <= 0 or amount ~= amount then
		return false
	end

	data.Coins += math.floor(amount)
	return true
end
```

## Specialist Habit

- Always define load, mutate, save, release.
- Always say what happens if load fails.
- Always separate persistent data from session-only cache.

## Deepin Merge

### Scope Additions

- Profile schema.
- Receipt idempotency.
- Economy reconciliation.
- Recovery flow.
- Backup and rollback.
- Purchase safety.

### Extra Standards

- Keep player data sacred.
- Read the schema and receipt patterns before writing data code.
- Favor safe migration over fast migration.
- Use one server-owned data service for all player mutations.
- Reconcile missing fields on load, not during gameplay.
- Save before teleport when the player changes server.
- Keep test data separate from live data.
- Use retry on save failure.

### Practical Rules

- Persistent data and session cache must stay separate.
- Every saved field needs a default and a version path.
- Every grant path needs an idempotency key.
- No non-serializable state in saved data.
- Open Cloud only when an external integration truly needs it.

### Extra Output Focus

- Mention schema version.
- Mention load failure behavior.
- Mention reconciliation behavior.
- Mention duplicate-grant prevention.
