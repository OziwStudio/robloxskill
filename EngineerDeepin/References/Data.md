# Data

## Purpose

- Use for save/load, ProfileStore/ProfileService, migration, OrderedDataStore, cross-server data, and data audits.
- Use `ImplementationPatterns.md` for schema, reconciliation, and receipt idempotency.

## Rules

- Player data is sacred.
- Use session locking for valuable production data.
- Use `UpdateAsync` over `SetAsync` when raw DataStore is used.
- Use schema versioning.
- Reconcile defaults.
- Validate before saving.
- Save on leave.
- Auto-save on interval.
- Use parallel saves in `BindToClose`.

## Schema

- Store only serializable data.
- Store ids, not Instances.
- Avoid nil fields.
- Use stable item ids.
- Split large data by category before size limit risk.
- Use dev prefix in Studio.
- Include receipt history when Developer Products exist.
- Include `DataVersion` or `_version`.
- Include recovery defaults for every field.

## Migration

1. Read `DataVersion`.
2. Run migrations sequentially.
3. Reconcile defaults.
4. Save current version.

## Lifecycle

1. Load profile.
2. Reconcile defaults.
3. Migrate old version.
4. Mark loaded player active.
5. Mutate through server services only.
6. Save on interval when dirty.
7. Save on leave.
8. Release session lock.

## Receipt Rule

- Use purchase id as idempotency key.
- Grant before returning `PurchaseGranted`.
- Return `NotProcessedYet` when profile is missing or grant fails.
- Never trust client purchase confirmation.
- Keep product config server-owned.

## Cross-Server

- MessagingService is temporary pub/sub.
- MemoryStore is temporary coordination.
- OrderedDataStore is integer leaderboard.
- DataStore is persistent state.

## Anti-Patterns

- Missing session locking.
- Saving every pickup.
- Not handling DataStore errors.
- Storing Instances.
- Sequential shutdown saves.
- Versionless profiles.
- Duplicate receipt grant.
- Client-side data mutation.
- Open Cloud key exposed to client.

## Concrete Patterns

- Profile schema: `ImplementationPatterns.md#Profile Schema`.
- Receipt safety: `ImplementationPatterns.md#Receipt Idempotency`.
- Remote validation: `ImplementationPatterns.md#Payload Validation`.
