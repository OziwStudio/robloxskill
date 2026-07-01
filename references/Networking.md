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
- Network ownership.
- Lag compensation.
- Streaming.
- Cross-server flow.
- Physics replication.
- Character replication.

## Activate When
- User mentions RemoteEvent, RemoteFunction, multiplayer, combat, trading, replication, latency, prediction, teleport, matchmaking, or cross-server flow.

## Standards
- Read `references/bridges/ImplementationPatterns.md` when remote map, rate limit, or payload validation code is needed.
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
- Keep networking responsive, secure, and bandwidth-efficient.
- Keep state synchronization predictable.
- Keep replication strategy explicit.
- Keep latency-sensitive actions server-validated.
- Keep movement and combat separate from cosmetic sync.

## Review Gate
- Who owns the state?
- Who receives updates?
- How often does it update?
- Can remote be spammed?
- Can payload be forged?
- Can latency desync the result?
- Is prediction justified?
- Does modern API path have stable fallback?
- Is ownership assigned correctly?
- Is replicated data necessary?
- Is sync frequency bounded?
- Is rate limit per action explicit?
- Is reconciliation path defined?
- Is physics ownership correct?

## Anti-Patterns
- Client Authority.
- Remote Spam.
- Massive Payload.
- Overused RemoteFunction.
- Excessive Replication.
- State Desync.
- Network Ownership Abuse.
- Unnecessary client trust.
- High-frequency blocking reads.
- Prediction without reconciliation.
- Sync spam.
- Large secret payloads.
- Unbounded sync frequency.
- Movement authority on the wrong peer.
- Blocking reads in hot paths.
- Mixing cosmetic traffic with valuable actions.

## Output Focus
- For systems: define remote map.
- For audit: list remote surface first.
- For performance: reduce frequency and payload.
- Include remote names, direction, payload, rate, and authority owner.
- Include replication owner and consumer.
- Include reconciliation behavior when prediction exists.
- Include sync frequency and bandwidth cost.

## Practical Principles
- One remote can handle multiple actions only when payloads remain clear.
- High-frequency visuals should not share remotes with valuable actions.
- RemoteFunction is for rare read-only queries or explicit blocking flows.
- UnreliableRemoteEvent is only for loss-tolerant updates.
- Server-to-client sync should send snapshots or deltas, never secret state.
- Cross-server messages should stay separate from persistent data.
- Physics ownership should match the actual movement owner.
- Character replication should stay server-safe.
- Use snapshots for stable state and deltas for frequent changes.
- Keep server-owned combat state separate from client presentation.
- Keep blocking reads rare and explicit.

## Remote Function Rules
- Use only for rare read-only or blocking flows.
- Avoid in hot combat or high-frequency loops.
- Keep the return shape small.
- Prefer remotes or async flows when possible.

## Remote Event Rules
- Give every remote a clear purpose.
- Keep one remote per stable action family when payloads stay clear.
- Do not put valuable state and cosmetic spam on the same remote.
- Keep event frequency bounded.

## Client-Server Architecture

### Client Responsibilities
- Visual effects.
- UI.
- Input processing.
- Local prediction.
- Presentation.

### Server Responsibilities
- Game authority.
- Data integrity.
- Economy systems.
- Progression systems.
- Validation.
- Security.

## Replication Architecture
- Replicate only needed state.
- Define data source, owner, consumer, and frequency.
- Avoid unnecessary information.
- Prefer snapshots or deltas over secret state.
- Keep replication predictable and controlled.
- Keep server-owned state authoritative even when mirrored.
- Replication should never become a hidden authority layer.

## Data Flow Analysis
- Identify data source.
- Identify data owner.
- Identify data consumers.
- Identify replication frequency.
- Keep data flow predictable.

## Server Authority
- Server owns damage, economy, inventory, and trading outcomes.
- Client requests actions.
- Server decides outcomes.
- Never trust client-provided results.

## Client Prediction
- Use prediction only when it improves responsiveness.
- Reconcile predicted state with the server.
- Avoid unnecessary complexity.
- Keep prediction optional and validated.
- Reconcile on mismatch immediately.
- Keep prediction local to the client when possible.

## Lag Compensation
- Use for hit registration, projectile systems, and melee only when justified.
- Balance fairness and accuracy.
- Consider network conditions.
- Do not use compensation as a replacement for server authority.
- Keep compensation bounded.
- Keep compensation validated against server state.

## Network Ownership
- Analyze physics ownership, character ownership, and vehicle ownership.
- Match ownership to the object that should move it.
- Avoid exploit-prone ownership assignments.
- Review replication behavior after changes.
- Keep ownership changes explicit.
- Recheck ownership when gameplay state changes.
- Do not hand out movement authority unnecessarily.

## Multiplayer Synchronization
- Keep state synchronization consistent.
- Keep position synchronization smooth.
- Keep combat synchronization validated.
- Keep animation synchronization bounded.
- Avoid excessive update rates.
- Sync only the minimum required state.
- Use lower rates for noncritical data.
- Keep animation sync cosmetic unless gameplay needs it.

## Competitive Networking
- Review latency fairness.
- Review hit validation.
- Review state consistency.
- Review spam resistance.
- Review sync safety under pressure.
- Keep competitive actions server-owned.
- Treat latency as a fairness issue, not just a performance issue.

## Cross-Server Flow
- Use MessagingService for transient cross-server messages.
- Use TeleportService for handoff.
- Keep server-to-server data separate from persistent storage.
- Keep arrival payloads small.
- Validate cross-server state on arrival.
- Keep server handoff data minimal.
- Separate transient messages from persistent data.

## Streaming Rules
- Consider StreamingEnabled for large worlds.
- Keep streamed state compatible with gameplay.
- Do not rely on content that may stream out.
- Keep important state authoritative on the server.
- Treat streamed-out content as unavailable until proven loaded.

## Bandwidth Rules
- Keep payloads small.
- Rate limit actionable remotes.
- Avoid unnecessary replication.
- Prefer fewer, clearer messages.
- Keep high-frequency visuals separate from valuable actions.
- Compress message shape when stable.
- Reduce update frequency before increasing payload complexity.

## Output Shape
```text
Remote Map:
Replication Owner:
Consumer:
Update Frequency:
Prediction:
Reconciliation:
Lag Compensation:
Ownership:
Risks:
```

## Audit Focus

- Use this file for remote surface review and replication boundaries.
- Treat remote payloads as the primary attack surface.
- Keep the client-to-server path intent-based.
- Keep server-to-client payloads minimal and non-sensitive.
- Prefer RemoteEvent over RemoteFunction unless blocking reads are unavoidable.
- Keep high-frequency gameplay traffic separate from cosmetic traffic.

## Audit Checks

- Every client-to-server remote has a clear owner, purpose, and validation gate.
- Every server-to-client sync path sends only necessary state.
- Every RemoteFunction has a strong reason to exist and a bounded call rate.
- Every high-frequency path has explicit frequency and payload limits.
- Replicated data does not expose secrets, internal IDs, or exploit-relevant configs.
- Network ownership matches the object that should move it.
- Prediction has reconciliation and does not replace server authority.

## Remote Contract Rules

- Client sends intent, not outcome.
- Server validates every payload before acting.
- Payloads stay small, stable, and schema-driven.
- Extra arguments are rejected.
- Rate limiting is per player and per action.
- Blocking calls stay rare and explicit.

## Remote Surface Review
- List every client to server remote.
- List every server to client sync path.
- List every blocking call.
- List every high-frequency message.
- List every cross-server handoff.

## Sync Cadence
- Keep combat sync bounded.
- Keep movement sync bounded.
- Keep UI state sync bounded.
- Keep noncritical replication slower than frame rate.
- Keep validation separate from sync.

## Prediction Review
- Ask whether prediction is necessary.
- Ask whether reconciliation is cheap enough.
- Ask whether the server remains authoritative.
- Ask whether the result is still fair under lag.

## Ownership Review
- Ask who moves the object.
- Ask who validates the object.
- Ask who reclaims ownership.
- Ask whether ownership changes create exploits.

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
