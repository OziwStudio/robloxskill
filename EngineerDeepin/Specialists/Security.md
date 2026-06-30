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

## Activate When

- User calls `!vulnfix`.
- User asks admin, permission, economy, inventory, trade, reward, damage, purchase, or anti-cheat.
- Any system has exploitable value.

## Standards

- Read `EngineerDeepin/References/ImplementationPatterns.md` when validation, rate limit, or server authority code is needed.
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

## Validation Gate

- Validate type.
- Validate range.
- Validate ownership.
- Validate permission.
- Validate cooldown.
- Validate state.
- Validate distance.
- Validate transaction.

## Exploit Questions

- How can attacker forge this?
- How can attacker repeat this?
- How can attacker race this?
- How can attacker bypass permission?
- How can attacker duplicate value?
- Can cloud scope be abused?

## Anti-Patterns

- Client currency.
- Client inventory.
- Trusting remote payload.
- Missing rate limit.
- Weak ownership check.
- Reward duplication.
- Trade race condition.
- Hidden security assumption.

## Output Focus

- `!vulnfix` starts with exploit path.
- Patch authority before polish.
- Keep false positives in mind for movement checks.
- Include concrete validation gates for each exploitable remote.

## Practical Principles

- Treat every remote as a public endpoint.
- Validate intent before value mutation.
- Deny silently for normal invalid input.
- Log repeated suspicious abuse, not every failed input.
- Prefer resetting impossible state over immediate kick.

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
