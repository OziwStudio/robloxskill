# Networking

## Purpose

- Use for remotes, multiplayer, teams, lobbies, rounds, matchmaking, TeleportService, MessagingService, and cross-server systems.

## Remote Rules

- Client sends intent.
- Server decides outcome.
- Validate type, range, ownership, state, cooldown.
- Rate limit per player.
- Keep payload small.
- Prefer RemoteEvent over RemoteFunction.
- Use UnreliableRemoteEvent only for loss-tolerant high-frequency data.

## Multiplayer Flow

- Handle `PlayerAdded`.
- Handle `PlayerRemoving`.
- Handle existing players in Studio.
- Character can be nil.
- Clean player state on leave.
- Validate player count before round start.

## Round Systems

- Use state machine.
- Common states: Intermission, Countdown, Playing, Results.
- Handle mid-round joins.
- Handle disconnects.
- Reset state cleanly.

## Teleport And Matchmaking

- Save before teleport.
- Wrap teleport in `pcall`.
- Handle reserved server access.
- Handle reconnect when useful.
- Remove disconnected players from queue.

## Cross-Server

- MessagingService is rate-limited pub/sub.
- Keep messages small.
- Prune stale server listings.
- Do not use MessagingService as persistence.

## Anti-Patterns

- Client score.
- Client match result.
- Unbalanced teams.
- Teleport without save.
- No cleanup on leave.
- Remote spam.
