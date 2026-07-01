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
- Read `references/ImplementationPatterns.md` when remote map, rate limit, or payload validation code is needed.
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

## Practical Principles
- One remote can handle multiple actions only when payloads remain clear.
- High-frequency visuals should not share remotes with valuable actions.
- RemoteFunction is for rare read-only queries or explicit blocking flows.
- UnreliableRemoteEvent is only for loss-tolerant updates.
- Server-to-client sync should send snapshots or deltas, never secret state.

## Remote Contract Example
```text
Remote: Inventory_Equip_RE
Direction: Client -> Server
Rate: 0.35s per player
Payload:
  ItemId: string 1..64
Validation:
  player loaded
  item exists in server inventory
  item is equippable
  character state allows equip
Outcome:
  server equips item
  server sends Inventory_Sync_RE
```

## Rate Gate Example
```luau
local lastCall: { [Player]: number } = {}
local function allow(player: Player, cooldown: number): boolean
	local now = os.clock()
	local last = lastCall[player]
	if last and now - last < cooldown then
		return false
	end
	lastCall[player] = now
	return true
end
```

## Specialist Habit
- Always draw the remote surface before writing remote handlers.
- Always state who owns the final value.
- Always include cleanup for per-player rate maps.

## Deepin Merge

### Scope Additions
- Remote map.
- Rate limiter.
- Payload validation.
- Remote handler.
- Server authority.
- Prediction.
- Sync frequency.
- Latency compensation.

### Extra Standards
- Read the remote map and validation patterns before coding network flows.
- Client sends intent only.
- Server decides the result.
- Every payload must be validated.
- High-frequency visuals must not share the same remote as valuable actions.
- Prefer small payloads and clear ownership.
- Keep prediction reconcile-required.
- Consider StreamingEnabled and network ownership when relevant.

### Practical Rules
- Define direction, payload, rate, and owner for every remote.
- Use RemoteFunction only for rare blocking reads.
- Use UnreliableRemoteEvent only for loss-tolerant updates.
- Send snapshots or deltas, never secret state.

### Extra Output Focus
- Show remote map.
- Show rate limit.
- Show payload shape.
- Show authority owner.
