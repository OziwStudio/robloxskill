# Production Gates

## Readiness

| Level | Meaning |
|---|---|
| `Not Ready` | unsafe, incomplete, or likely to fail |
| `Partially Ready` | works but needs hardening |
| `Production Ready` | safe, maintainable, tested by reasoning |
| `Enterprise Ready` | scalable for large live-service growth |

## Hard Blocks

- Client controls currency.
- Client controls inventory.
- Client controls rewards.
- Client controls damage.
- Client controls progression.
- Remote has no validation.
- Action remote has no rate limit.
- Receipt can grant twice.
- Data save can overwrite newer session.
- Connection leak grows forever.

## Sharp Edges

- DataStore needs session locking for valuable data.
- Client-side currency is forbidden.
- ProcessReceipt must be idempotent.
- BindToClose has limited time.
- RemoteEvent needs per-player rate limit.
- RemoteFunction can hang flows.
- `wait`, `spawn`, and `delay` are not preferred.
- Module require can deadlock through cycles.
- `#table` breaks with nil gaps.
- `WaitForChild` needs timeout in critical paths.
- Luau patterns are not regex.
- StreamingEnabled can hide instances.
- Network ownership affects physics authority.
- Mobile part count and UI density matter.

## Sharp Edge Severity

| Severity | Fix Before |
|---|---|
| `Critical` | shipping |
| `High` | current sprint |
| `Medium` | scale |
| `Low` | convenient cleanup |

## Critical Sharp Edges

- DataStore without session locking.
- Client-side currency or rewards.
- ProcessReceipt without idempotency.

## High Sharp Edges

- Undisconnected event connections.
- RemoteEvent flooding.
- Sequential BindToClose saves.

## Medium Sharp Edges

- Too many visible mobile parts.
- Yielding in module body.
- Nil gaps with `#table`.
- Infinite yield from `WaitForChild`.

## Low Sharp Edges

- Legacy `wait`, `spawn`, `delay`.
- Regex syntax used as Luau pattern.

## Final Code Gate

- Correct path.
- Correct script type.
- Correct require path.
- Correct remote name.
- Correct authority.
- Correct validation.
- Correct cleanup.
- Correct version.
- Correct header.
- Correct footer.
- Correct comment language.

## Security Gate

- Validate type.
- Validate range.
- Validate ownership.
- Validate cooldown.
- Validate state.
- Validate distance when spatial.
- Validate permission when privileged.
- Never trust client result.

## Data Gate

- Load safe.
- Reconcile schema.
- Migrate version.
- Save retry.
- Handle failure.
- Prevent duplicate grant.
- Separate session state from persistent state.

## Performance Gate

- Avoid busy loops.
- Avoid unbounded tables.
- Avoid repeated `GetDescendants` in hot paths.
- Avoid per-frame server work unless required.
- Disconnect events.
- Reuse instances when reasonable.
- Keep payloads small.
- Profile before heavy optimization.
- MeshPart collision fidelity is simplified.
- High-poly meshes have render fidelity or LOD plan.
- Mobile map density is reviewed.

## Modern API Gate

- Verify API availability.
- Verify project settings.
- Verify target platform.
- Verify permissions and scopes.
- Provide stable fallback.
- Do not expose cloud credentials.
- Do not replace server validation with modern API assumptions.
- Flag uncertain API as `Verify-Current`.

## Launch Gate

- FTUE first action exists.
- First reward exists.
- Mobile manual test passes.
- Multiplayer test passes when relevant.
- Purchase flow test passes.
- Data leave/rejoin test passes.
- Remote spam test passes.
- No critical Output errors.
- Community safety assumptions are marked when social, chat, ads, or UGC exists.
