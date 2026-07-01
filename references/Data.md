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
- Inventory persistence.
- Backup and rollback.
- Schema reconciliation.
- Load failure handling.
- Cross-server data.
- OrderedDataStore.

## Activate When

- User asks data, save, load, profile, inventory, economy, purchase, receipt, leaderboard, migration, backup, rollback, or duplication.

## Standards

- Read `references/bridges/ImplementationPatterns.md` when schema, migration, or receipt examples are needed.
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
- Use one server-owned data service for mutations.
- Keep serializable state only.
- Keep temporary state out of saved data.
- Use atomic save paths where possible.

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
- Can load fail safely?
- Can schema migrate safely?
- Can recovery restore a safe point?
- Can cross-server messages be confused with storage?

## Anti-Patterns

- Missing session lock.
- Unsafe save.
- Save spam.
- Versionless profile.
- Massive profile.
- Duplicate save path.
- Race condition.
- Missing recovery.
- Missing rollback.
- Missing schema version.
- Missing persistent/session separation.
- Missing safe load fallback.
- Missing backup point.

## Output Focus

- For persistence: define schema and lifecycle.
- For receipt: define idempotency.
- For audit: lead with data-loss risk.
- Include schema version, reconciliation, save lifecycle, and failure behavior.
- Include load, mutate, save, release, and recovery.
- Include migration path and rollback path.

## Practical Principles

- Data changes should pass through one server-owned data service.
- Every profile field needs a default and migration story.
- Save frequency should be dirty-state based, not every event.
- Receipts need an idempotency key before granting value.
- Never store Instances, Vector3 userdata, or non-serializable objects directly.
- Keep save and session state separate.
- Save on leave and shutdown.
- Retry failed saves carefully.
- Reconcile missing fields on load.

## Data Ownership
- Every field needs an owner.
- Every field needs a mutation rule.
- Every field needs a lifecycle.
- Persistent data lives across sessions.
- Session data lives only in memory.

## Persistence Architecture
- Choose raw DataStore only for simple cases.
- Use ProfileStore or equivalent for production safety.
- Use a wrapper only when the project truly needs custom behavior.
- Use `UpdateAsync` for atomic persistence when possible.
- Use `RemoveAsync` carefully and intentionally.

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

## Persistence Reference

### Overview

Data persistence means player progress survives across sessions.
The server loads data when the player joins, keeps it in memory during play, and saves it back on leave or on a safe interval.

### Data Architecture Choices

| Approach | Best For | Complexity |
|---|---|---|
| Raw DataStoreService | simple games, prototypes | low |
| ProfileService | production games, session safety | medium |
| Custom wrapper | specific advanced requirements | high |

### Core DataStore Methods

| Method | Purpose | Notes |
|---|---|---|
| `GetDataStore(name)` | get or create a named store | returns a DataStore object |
| `GetAsync(key)` | read a value | returns nil if key does not exist |
| `SetAsync(key, value)` | overwrite a value | no conflict protection |
| `UpdateAsync(key, callback)` | atomic read-modify-write | preferred for saves |
| `RemoveAsync(key)` | delete a key | use carefully |

`UpdateAsync` is preferred because it prevents race conditions.

### Save and Load Flow

1. player joins
2. server loads data from DataStore
3. server populates in-memory state
4. player plays
5. server autosaves periodically
6. player leaves
7. server saves and clears cache
8. server shutdown triggers final save

## Load Architecture
- Validate the loaded data.
- Reconcile missing fields.
- Recover from schema mismatch.
- Handle missing profile safely.
- Never assume old data already matches the new code.

## Save Architecture
- Save on a safe interval.
- Save on `PlayerRemoving`.
- Save on `BindToClose`.
- Avoid save spam.
- Retry failed saves.
- Use dirty-state based saves when possible.

## Recovery Architecture
- Define a restore point.
- Keep rollback plans ready.
- Restore from known good state when corruption appears.
- Treat recovery as a production feature, not a last resort.

## Backup Systems
- Keep snapshot strategies.
- Keep emergency restoration paths.
- Keep a previous known-good state available when practical.
- Test recovery before you need it.

### Enable API Services

Studio testing requires API Services enabled in game settings.
Without that, DataStore calls fail in Studio.

### Leaderstats Pattern

- the leaderboard folder must be named `leaderstats`
- use `IntValue` for numbers and `StringValue` for text
- leaderstats are visible to everyone
- never put sensitive data in leaderstats

### ProfileService

ProfileService is the production-standard wrapper for persistence.
It solves session locking, autosave, migration, and corruption risks that raw DataStore usage leaves manual.

Rules:
- use it when the project needs production persistence
- reconcile data against a template
- release the profile on player leave
- handle the case where the profile loads on another server

### Session Locking
- Only one server owns a profile at a time.
- Release the lock on leave or shutdown.
- Avoid stale overwrite from old servers.
- Treat lock ownership as mandatory for valuable data.

### Session Locking

Session locking prevents two servers from saving stale copies over each other.

Rules:
- only one server owns the profile at a time
- the lock must be released on leave or release
- stale sessions must not overwrite fresh state

### Schema Design

- include a version number
- use defaults for new fields
- keep types stable
- store serializable values only
- never save Instances or Roblox userdata directly
- include explicit versioning
- keep defaults in a template
- keep types stable across versions
- keep temporary state separate from persistent state

### Schema Contract
```luau
export type PlayerData = {
	DataVersion: number,
	Coins: number,
	EquippedItemId: string?,
	Inventory: { [string]: number },
	Receipts: { [string]: boolean },
}
```

### Migration

When schema changes, migrate old data forward in steps.

Rules:
- read the version
- apply migrations in order
- save the new version back
- keep the code backward compatible
- keep migrations small and reversible
- preserve old data whenever possible

## Migration Engineering
- Plan schema changes before shipping them.
- Add new versions rather than mutating old ones blindly.
- Track migration order.
- Roll forward safely.
- Keep rollback possible when a change fails.

### OrderedDataStore

Use OrderedDataStore for leaderboards.

Rules:
- only integer values are supported
- update at reasonable intervals
- do not write every frame
- keep leaderboard writes bounded
- separate gameplay currency from leaderboard values when needed

## Economy Persistence
- Store currency consistently.
- Store purchase records.
- Store transaction history when needed.
- Protect against duplicate grant.
- Reconcile balance changes on load.

## Inventory Persistence
- Store item ownership, not instance objects.
- Store equipment state separately when helpful.
- Prevent item duplication.
- Prevent inventory corruption.
- Keep item identifiers stable.

## Progression Persistence
- Store experience, levels, unlocks, and achievements.
- Keep progression accurate across reconnects.
- Rebuild derived state on load.

## Cross-Server Data
- Use MessagingService for transient messages.
- Use DataStore for persistent shared state.
- Do not confuse broadcast with storage.
- Keep message payloads small.
- Keep storage writes intentional.

### Cross-Server Data

- use MessagingService for transient cross-server messages
- use DataStore for persistent cross-server state
- do not confuse broadcast with storage

### Best Practices

- autosave on interval
- save on PlayerRemoving
- save on BindToClose
- retry failed saves
- validate data before saving
- keep persistent data separate from session cache
- reconcile missing fields on load
- keep one server-owned mutation path
- save before teleport when server hopping
- use Studio-only test prefixes
- keep recovery and rollback ready

## Load Failure Handling
- Return a safe fallback when load fails.
- Do not invent data on failure unless the design says so.
- Let the player rejoin when needed.
- Keep the failure mode visible in logs.

## Output Focus
- Mention schema version.
- Mention load failure behavior.
- Mention reconciliation behavior.
- Mention duplicate-grant prevention.

### Anti-Patterns

- saving too frequently
- ignoring pcall
- storing Instances
- versionless schema
- missing session lock
- duplicate save paths
- relying on TeleportData for real state

### Sharp Edges

- DataStore is eventually consistent
- BindToClose has a time limit
- Studio testing can fail without API access
- key names are case-sensitive
- NaN and invalid numeric values can break saves

### Output Rule

- define schema first
- define load and save lifecycle
- mention session locking
- mention migration path
- mention failure behavior
