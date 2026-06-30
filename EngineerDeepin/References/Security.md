# Security

## Purpose

- Use for exploit prevention, remote validation, economy, inventory, trade, combat, admin, and purchase safety.
- Use `ImplementationPatterns.md` for validator, rate limiter, remote handler, and authority skeleton.

## Trust Model

- Client is untrusted.
- Remote payload is attacker-controlled.
- UI state is not authority.
- Server owns valuable outcomes.

## Remote Validation

- Type.
- Range.
- Ownership.
- Permission.
- Cooldown.
- State.
- Distance.
- Transaction.

## Validation Helper Pattern

- Validate at remote boundary.
- Return early on invalid input.
- Keep denial silent unless debug is needed.
- Log exploit pattern only when useful.
- Use `Payload Validation` pattern for primitives.
- Use `Rate Limiter` pattern for actionable remotes.
- Use `Remote Handler` pattern for server-owned purchases/actions.

## Protected Domains

- Currency.
- Inventory.
- Rewards.
- Damage.
- Progression.
- Trading.
- Purchases.
- Admin commands.

## Audit Steps

1. List remotes.
2. Trace value grants.
3. Find client authority.
4. Check validation.
5. Check rate limits.
6. Check data exposure.
7. Patch exploit path.

## Remote Hardening Flow

1. Name the remote.
2. Name the player intent.
3. Define payload schema.
4. Define cooldown.
5. Validate type and range.
6. Validate ownership and state.
7. Validate distance when spatial.
8. Apply server outcome only.
9. Save persistent result through data layer only.

## Admin Rule

- Do not hardcode a personal UserId in generated code.
- Ask user for owner/admin UserId.
- Keep permission tables server-only.
- Use role config, group rank, or user-provided UserId.

## Concrete Patterns

- Rate limit: `ImplementationPatterns.md#Rate Limiter`.
- Payload validation: `ImplementationPatterns.md#Payload Validation`.
- Remote handler: `ImplementationPatterns.md#Remote Handler`.
- Combat authority: `ImplementationPatterns.md#Server Authority Gameplay`.

## Anti-Patterns

- Client currency.
- Client damage.
- Client purchase grant.
- Missing rate limit.
- Weak ownership check.
- Duplicate reward path.
- Trade race condition.
- Admin UserId hardcoded without user-provided owner.
- Client-only anti-cheat as authority.
- Security hidden inside UI.
