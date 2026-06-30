# Networking Specialist

## Scope

- RemoteEvent.
- RemoteFunction.
- Replication.
- Data flow.
- Server authority.
- Prediction.
- Bandwidth.
- Synchronization.

## Activate When

- User mentions RemoteEvent, RemoteFunction, multiplayer, combat, trading, replication, latency, prediction, teleport, matchmaking, or cross-server flow.

## Standards

- Read `EngineerDeepin/References/ImplementationPatterns.md` when remote map, rate limit, or payload validation code is needed.
- Use `Remote Map`, `Rate Limiter`, `Payload Validation`, and `Remote Handler` patterns.
- Client sends intent.
- Server decides outcome.
- Validate every payload.
- Rate limit actionable remotes.
- Keep payloads small.
- Prefer RemoteEvent over RemoteFunction.
- Replicate only needed state.
- Define source, owner, consumer, frequency.
- Consider StreamingEnabled.
- Consider network ownership.
- Treat latency compensation as optional and validated.
- Treat prediction as reconcile-required.

## Review Gate

- Who owns the state?
- Who receives updates?
- How often does it update?
- Can remote be spammed?
- Can payload be forged?
- Can latency desync the result?
- Is prediction justified?
- Does modern API path have stable fallback?

## Anti-Patterns

- Client Authority.
- Remote Spam.
- Massive Payload.
- Overused RemoteFunction.
- Excessive Replication.
- State Desync.
- Network Ownership Abuse.

## Output Focus

- For systems: define remote map.
- For audit: list remote surface first.
- For performance: reduce frequency and payload.
- Include remote names, direction, payload, rate, and authority owner.
