# Architecture

## Purpose

- Pakai untuk service hierarchy, script placement, module boundary, framework choice, and file tree.
- Architecture must follow system complexity.
- Pakai `ImplementationPatterns.md` when architecture needs direct module or remote code.

## Service Placement

- `ServerScriptService`: server logic, data, anti-cheat, authoritative systems.
- `ReplicatedStorage`: shared modules, remotes, shared configs, shared assets.
- `StarterPlayerScripts`: client controllers, input, camera, local UI logic.
- `StarterGui`: ScreenGui templates.
- `ServerStorage`: server-only maps, tools, templates, private assets.
- `Workspace`: live world only; keep lean.

## Communication

- RemoteEvent: async client/server messaging.
- RemoteFunction: request/response only when blocking is acceptable.
- BindableEvent: same-side decoupled communication.
- UnreliableRemoteEvent: high-frequency loss-tolerant data only.

## Module Rules

- Use modules for reusable logic.
- Keep module body non-yielding.
- Use explicit `Init` and `Start` lifecycle.
- Use explicit `Destroy` lifecycle for cleanup-heavy services.
- Avoid circular requires.
- Centralize remotes.
- Use types and constants for contracts.

## Folder Scale

- Small game: flat but clean.
- Medium game: modular domains.
- Large game: framework or service/controller structure only when justified.

## Complexity Classifier

| Class | Use |
|---|---|
| `Simple` | isolated server-only feature, one script is enough |
| `Moderate` | server + client + remotes + shared config |
| `Complex` | persistent data, multiple services, UI, economy, or live-service risk |

## Moderate System Tree

```text
ReplicatedStorage/
  SystemName/
    Remotes/
    Modules/
    ConfigSystem.luau

ServerScriptService/
  SystemServer/
    Main.server.luau
    Services/
    Handlers/

StarterPlayerScripts/
  SystemClient/
    Main.local.luau
    Controllers/
```

## Boundary Rule

- Server owns rules and persistent state.
- Client owns input and presentation.
- Shared owns types, constants, pure helpers, and remote contracts.
- Workspace owns live physical objects only.
- ServerStorage owns private templates and assets.

## Anti-Patterns

- God scripts.
- Server logic in shared storage.
- Scripts in Workspace for core logic.
- Circular requires.
- Polling instead of events.
- Overusing RemoteFunctions.
- Framework added before complexity justifies it.
- Shared module mutating server-only state.

## Concrete Patterns

- Service module: `ImplementationPatterns.md#Module Shape`.
- Remote contract: `ImplementationPatterns.md#Remote Map`.
- Remote boundary: `ImplementationPatterns.md#Remote Handler`.
