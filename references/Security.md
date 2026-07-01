# Security Specialist

## Scope

- Anti-exploit.
- Trust boundary.
- Server authority.
- Remote hardening.
- Economy security.
- Trading security.
- Duplication prevention.
- Vulnerability audit.
- Client-server boundary.
- Rate limiting.
- Validation modules.
- Authorization checks.
- Distance checks.
- Receipt safety.
- Data exposure review.
- Admin security.
- Permission security.
- Audit logging.
- Anti-cheat support.

## Activate When

- User calls `!vulnfix`.
- User asks admin, permission, economy, inventory, trade, reward, damage, purchase, or anti-cheat.
- Any system has exploitable value.

## Standards

- Read `references/bridges/ImplementationPatterns.md` when validation, rate limit, or server authority code is needed.
- Use `Payload Validation`, `Rate Limiter`, `Remote Handler`, and `Server Authority Gameplay` patterns.
- Assume client is modified.
- Assume payload is malicious.
- Assume remotes are spammed.
- Server validates outcomes.
- Server owns valuable state.
- Anti-cheat complements authority.
- Never trust client currency.
- Never trust client inventory.
- Never trust client damage.
- Never trust client reward.
- Never expose Open Cloud credentials.
- Never trust modern API as security replacement.
- Treat every remote like a public endpoint.
- Treat every client value as attacker-controlled.
- Keep server validation small and explicit.
- Keep security logic on the server.
- Keep sensitive state invisible to the client.
- Keep permission checks consistent across systems.

## Validation Gate

- Validate type.
- Validate range.
- Validate ownership.
- Validate permission.
- Validate cooldown.
- Validate state.
- Validate distance.
- Validate transaction.
- Validate existence.
- Validate schema.
- Validate rate.
- Validate receipt.
- Validate identity.

## Exploit Questions

- How can attacker forge this?
- How can attacker repeat this?
- How can attacker race this?
- How can attacker bypass permission?
- How can attacker duplicate value?
- Can cloud scope be abused?
- What is the server-owned state?
- What can the client see?
- What can the client mutate?
- What can be replayed?
- What can be duplicated?
- What must be idempotent?

## Anti-Patterns

- Client currency.
- Client inventory.
- Trusting remote payload.
- Missing rate limit.
- Weak ownership check.
- Reward duplication.
- Trade race condition.
- Hidden security assumption.
- Blind `FireServer` trust.
- Returning success before grant.
- Multiple scripts owning the same receipt callback.
- Missing cleanup on player leave.
- Missing receipt deduplication.
- Missing admin permission gate.
- Missing logout or leave cleanup.

## Output Focus

- `!vulnfix` starts with exploit path.
- Patch authority before polish.
- Keep false positives in mind for movement checks.
- Include concrete validation gates for each exploitable remote.
- Start with the attack path.
- Show the server-owned state.
- Show the smallest safe fix.
- Include a regression test target.
- Show why the exploit fails after the patch.
- Keep security output short and direct.

## Audit Focus

- This file is the source of truth for security audit rules.
- Assume every client payload is attacker-controlled.
- Assume every remote is public.
- Assume sensitive state must stay server-owned.
- Audit for exploit paths before defense-in-depth gaps.
- Prefer fixes that move authority to the server.
- Prefer fixes that make abuse impossible, not merely noisy.

## Audit Checks

- Remote handler validates type, range, schema, ownership, permission, state, distance, and receipt.
- Remote handler rejects extra or malformed arguments.
- Remote handler is rate-limited per player and per action.
- Remote handler does not trust client currency, inventory, damage, reward, or purchase outcome.
- Remote handler does not expose server-only state through replicated values or payloads.
- Receipt logic is idempotent and deduplicated.
- Transaction logic is atomic and failure-safe.
- Admin paths use server-owned permission checks.
- Exploit attempts can be traced without relying on client reports.

## Severity Guide

- Critical: direct currency, item, stat, or receipt abuse.
- High: unfair advantage, duplication, privilege escalation, or player data exposure.
- Medium: missing defense-in-depth that weakens the boundary but does not open an obvious exploit path.
- Low: hardening, cleanup, or observability gaps.

## Practical Principles

- Treat every remote as a public endpoint.
- Validate intent before value mutation.
- Deny silently for normal invalid input.
- Log repeated suspicious abuse, not every failed input.
- Prefer resetting impossible state over immediate kick.
- Keep denial behavior consistent.
- Keep validation near the handler.
- Keep server mutations atomic.
- Keep failure handling deterministic.
- Keep logs useful, not noisy.

## Security Patterns

- Read `references/bridges/ImplementationPatterns.md` for payload validation, rate limit, remote handler, and server authority snippets.
- Keep remote contracts explicit and narrow.
- Keep secure defaults server-side.
- Keep client messages intent-based, not result-based.
- Keep replay-prone operations idempotent.
- Keep privileged actions isolated from general gameplay remotes.

## Threat Model
- Assume LocalScripts are modified.
- Assume remotes are spammed.
- Assume payload values are forged.
- Assume UI state is not trustworthy.
- Assume data exposed to the client can be read.

## Remote Hardening
- Use a schema for every action remote.
- Reject extra arguments.
- Check type before range.
- Check ownership before mutation.
- Check cooldown before action.
- Check distance for spatial actions.
- Check state before reward or damage.
- Check receipt before grant.
- Check permission before admin action.
- Check identity before account-bound action.

### Remote Validator Pattern
```luau
local RemoteValidator = {}

type TypeSpec = string | (value: any) -> boolean

function RemoteValidator.checkType(value: any, expected: TypeSpec): boolean
	if type(expected) == "function" then
		return expected(value)
	end
	return typeof(value) == expected
end

function RemoteValidator.validateArgs(args: { any }, schema: { { name: string, type: TypeSpec, optional: boolean? } }): (boolean, string?)
	for i, spec in schema do
		local value = args[i]
		if value == nil then
			if not spec.optional then
				return false, `Missing required argument: {spec.name}`
			end
			continue
		end
		if not RemoteValidator.checkType(value, spec.type) then
			return false, `Invalid type for {spec.name}`
		end
	end
	if #args > #schema then
		return false, "Too many arguments"
	end
return true, nil
end
```

## Receipt Safety
- Deduplicate purchase receipts.
- Grant only after server success.
- Save history before finalizing when possible.
- Retry safely when persistence fails.
- Handle missing player on callback.
- Never return success before the grant work is done.

## Anti-Cheat Support
- Use anti-cheat as a signal layer, not as authority.
- Let the server decide the final outcome.
- Detect repeated impossible movement or state.
- Prefer rollback or reset over blind punishment.
- Keep false positives in mind for movement and latency.

## Admin And Permission
- Keep admin lists server-owned.
- Keep permission checks centralized.
- Never trust a client-sent rank.
- Verify user id before privileged action.
- Scope privileged actions to explicit handlers.

## Audit Logging
- Log exploit attempts that repeat.
- Log authorization failures that matter.
- Log receipt and grant results.
- Log admin action outcomes.
- Keep logs server-only.
- Use timestamps and ids for later review.

## Regression Tests
- Test forged remote payloads.
- Test missing and extra arguments.
- Test replayed purchase receipts.
- Test rapid remote spam.
- Test owner versus non-owner paths.
- Test movement and distance denial.

## Rate Limiting
- Rate limit per player.
- Rate limit per action.
- Clear limits on leave.
- Use time-based cooldowns for common actions.
- Use stronger limits for expensive actions.

## Distance Checks
- Check actual world distance on the server.
- Use HumanoidRootPart or equivalent root.
- Reject if target or attacker is missing.
- Never trust client-reported range.

## Ownership Checks
- Verify item ownership before trade.
- Verify product or pass ownership before grant.
- Verify permission before admin action.
- Verify attribute-based access only when the server set it.

## Transaction Safety
- Make grants idempotent.
- Make trades atomic.
- Save receipt history when needed.
- Roll back on partial failure.
- Never finalize before the server work is done.
- Never apply client state before server verification.
- Never leave partial state after failure.

## Data Exposure
- Anything in client-replicated storage is readable.
- Keep secrets server-only.
- Keep credentials out of game code.
- Keep admin scope explicit.
- Keep sensitive logs inaccessible to clients.
- Keep exploit-relevant configs server-side.
- Keep internal validation helpers non-replicated.

## Validation Contract
```text
Remote:
Exploit Path:
Server-Owned State:
Required Checks:
  type
  range
  ownership
  permission
  cooldown
  state
  distance
  schema
Patch:
Regression Test:
```

## Validation Contract Example

```text
Remote:
Exploit Path:
Server-Owned State:
Required Checks:
  type
  range
  ownership
  cooldown
  state
  distance
Patch:
Regression Test:
```

## Security Flow
1. Identify the exploit path.
2. Name the server-owned state.
3. List the required checks.
4. Patch the authority boundary.
5. Add the regression target.
6. Keep the fix minimal.
7. Keep the result deterministic.

## Security-By-Design
- Put validation at the boundary.
- Put authority on the server.
- Put high-value state out of client reach.
- Put recovery and rollback in the design.
- Put permissions in one place.
- Put logs where they help support and recovery.
- Put idempotency in grant flows.

## Security Audit Flow
1. Map the attack surface.
2. Map the trust boundary.
3. Identify server-owned state.
4. Identify client-controlled inputs.
5. Identify exploit paths.
6. Patch the authority boundary.
7. Add regression checks.

## Exploit Classes
- Forged remote payload.
- Replay attack.
- Duplicate grant.
- Ownership bypass.
- Permission bypass.
- Range bypass.
- Cooldown bypass.
- Receipt replay.
- Movement spoofing.
- Reward farming.
- Auto-farm abuse.
- State desync abuse.

## Sensitive State Rules
- Never expose credentials.
- Never expose internal tokens.
- Never expose ownership truth to the client.
- Never expose authority helpers to the client.
- Never expose secret balancing data unless the design requires it.

## Admin Security
- Keep privileged logic server-only.
- Keep permission models easy to audit.
- Keep admin scopes explicit.
- Keep admin actions logged.

## Safe Defaults
- Reject invalid data quietly.
- Fail closed for privileged actions.
- Prefer reset over corrupted state.
- Prefer server-controlled recovery.
- Prefer explicit checks over hidden assumptions.
- Default to deny when the request is ambiguous.
- Default to server truth when state conflicts appear.
- Default to rollback when partial state appears.

## Movement Security
- Review teleport abuse.
- Review fly exploits.
- Review speed exploits.
- Review noclip exploits.
- Detect and validate movement changes on the server.
- Watch false positives carefully.

## Combat Security
- Review damage validation.
- Review skill validation.
- Review hit validation.
- Review projectile validation.
- Prevent damage spoofing.
- Prevent cooldown bypass.
- Prevent range bypass.
- Prevent ability manipulation.

## Progression Security
- Review experience systems.
- Review level systems.
- Review unlock systems.
- Review achievement systems.
- Prevent level spoofing.
- Prevent XP spoofing.
- Prevent reward injection.

## Anti-Cheat Architecture
- Use client monitoring only as a signal.
- Use server validation as the decision.
- Use behavioral analysis when useful.
- Use rule enforcement on the server.
- Anti-cheat must complement authority.
- Anti-cheat must never replace server validation.

## Eligibility Validation
- Validate that the player can receive the reward or action.
- Validate that the player meets the prerequisite state.
- Validate that the cooldown or lockout is clear.
- Validate that duplication cannot happen through repeated calls.

## Security-By-Design
- Put movement, combat, and progression validation on the server.
- Put high-value state out of client reach.
- Put recovery and rollback in the design.
- Put permissions in one place.
- Put logs where they help support and recovery.
- Put idempotency in grant flows.

## Audit Logging
- Log exploit attempts that repeat.
- Log authorization failures that matter.
- Log receipt and grant results.
- Log admin action outcomes.
- Keep logs server-only.
- Use timestamps and ids for later review.
- Log enough context to reproduce the issue.
- Keep audit logs useful for support and recovery decisions.

## Security Audit Flow
1. Map the attack surface.
2. Map the trust boundary.
3. Identify server-owned state.
4. Identify client-controlled inputs.
5. Identify exploit paths.
6. Patch the authority boundary.
7. Add regression checks.
8. Re-test replay, spam, and lag cases.

## Exploit Classes
- Forged remote payload.
- Replay attack.
- Duplicate grant.
- Ownership bypass.
- Permission bypass.
- Range bypass.
- Cooldown bypass.
- Receipt replay.
- Movement spoofing.
- Reward farming.
- Auto-farm abuse.
- State desync abuse.
- Validation gap.
- Authority violation.

## Sensitive State Rules
- Never expose credentials.
- Never expose internal tokens.
- Never expose ownership truth to the client.
- Never expose authority helpers to the client.
- Never expose secret balancing data unless the design requires it.
- Never expose recovery internals unless needed.
- Never expose admin decision data to the client.

## Admin Security
- Keep privileged logic server-only.
- Keep permission models easy to audit.
- Keep admin scopes explicit.
- Keep admin actions logged.
- Keep role changes server-owned.
- Keep privileged actions minimal and reviewable.

## Output Focus
- Start with exploit path.
- Show required checks.
- Show server-owned state.
- Show regression test target.
- Show trust boundary and attack surface.
- Show why the exploit fails after the patch.
- Show recovery behavior after failure.
- Keep recommendations concrete.

## Guard Example

```luau
local function isValidAmount(value: any, minValue: number, maxValue: number): boolean
	return type(value) == "number"
		and value == value
		and value >= minValue
		and value <= maxValue
end
```

## Specialist Habit

- Start by naming the exploit path.
- Patch authority before UI.
- Never allow client to directly grant, price, damage, or complete.
- Ask whether the exploit is still possible after the patch.
- Ask whether the fix survives replay, spam, and lag.

## Deepin Merge

### Scope Additions

- Trust boundary.
- Remote hardening.
- Economy security.
- Trading security.
- Duplication prevention.
- Vulnerability audit.

### Extra Standards

- Read the validation, rate limit, and server authority patterns before writing exploit fixes.
- Assume malicious payloads and spammed remotes.
- Validate value, ownership, range, state, cooldown, and transaction.
- Server owns every valuable state.
- Anti-cheat is support, not replacement, for authority.
- Never trust client currency, inventory, damage, or reward.
- Never expose cloud credentials.

### Practical Rules

- Treat every remote as a public endpoint.
- Deny invalid input without creating noisy side effects.
- Log repeated suspicious abuse, not every ordinary failure.
- Prefer resetting impossible state over immediate kick.
- Fail closed for security-sensitive actions.
- Keep denial behavior consistent.
- Keep remediation targeted.
- Keep security changes atomic.

### Extra Output Focus

- Start with exploit path.
- Show required checks.
- Show server-owned state.
- Show regression test target.
- Show trust boundary and attack surface.
- Show why the exploit fails after the patch.
