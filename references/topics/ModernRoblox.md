# Modern Roblox

## Purpose

- This reference adopts modern Roblox API awareness strengths.
- Use only when Deepin is active and the user asks for modern APIs, LiveOps, modern avatar, modern UI, Open Cloud, streaming, or publish readiness.
- Do not assume the newest API is always available.
- Verify an API before making it a hard requirement.

## API Confidence

| Level | Rule |
|---|---|
| `Stable` | may be used directly when it fits the need |
| `Project-Enabled` | use when the project already enables it or the user requests it |
| `Verify-Current` | check current documentation/API before use |
| `Fallback-Required` | provide a stable alternative |

## Modern Decision Flow

1. Identify target platform.
2. Identify project age: greenfield or brownfield.
3. Identify enabled services.
4. Classify API confidence.
5. Choose stable path first.
6. Add modern path only when useful.
7. Add fallback when API is uncertain.
8. State assumptions briefly.

## 2026 Coverage Map

| Area | Default | Modern Path | Fallback |
|---|---|---|---|
| Luau typing | typed modules where useful | new type solver features | `--!strict` or nonstrict with runtime validation |
| Server authority | server owns outcomes | server authority with latency compensation when available | manual validation and reconciliation |
| Streaming | handle missing instances | frustum or multi-focus streaming when enabled | conservative StreamingEnabled assumptions |
| UI | reusable components | UI Stylesheets and styling transitions when available | traditional ScreenGui components |
| Input | device-aware layouts | `PreferredInput` and viewport-aware routing when available | UserInputService and GuiService checks |
| Avatar | legacy Humanoid fallback | AvatarAbilities-style controller libraries when project-enabled | Humanoid state and custom controllers |
| Open Cloud | engine services first | Open Cloud Data Stores and Configs for external tooling | DataStoreService and local config modules |
| LiveOps | safe defaults | cloud config, analytics, flags | versioned ModuleScript config |
| Audio | SoundService manager | modern audio emitter/listener paths when verified | basic Sound objects |
| Moderation | Roblox safety first | content moderation and social APIs when verified | restrict UGC and require manual approval |
| Monetization | server receipt grant | subscriptions or newer commerce APIs when verified | Developer Products and Game Passes |
| Serialization | strict adapters | external JSON contract adapters | explicit encode/decode validation |

## Verify-Current APIs

- Individual `UICorner` radius properties.
- `UIShadow`.
- Studio Debugger Luau API.
- Roblox Open Cloud MCP integrations.
- `PromptRobuxTransferAsync`.
- `game.ServerRestartScheduled`.
- LibMP or MicroProfiler Luau access.
- `PlayerOwnsAssetAsync` privacy behavior.
- `TextChatService:GetChatGroupsAsync()`.
- `VoiceChatService:GetVoiceGroupsAsync()`.
- `AutoLocalize` behavior for Text UI.
- Removed Roblox Connect Calling APIs.
- New type solver advanced behavior.
- Server authority with latency compensation.
- UI Stylesheets.
- Styling transitions.
- ViewportDisplaySize.
- PreferredInput.
- AvatarAbilities libraries.
- DragDetectors behavior in target context.
- Open Cloud Configs.
- Open Cloud Data Stores scopes.
- Subscriptions in Robux.
- Acoustic simulation.
- AudioEmitter and AudioListener behavior.
- Content moderation APIs.
- Trusted connections and age group APIs.
- Moments or social sharing APIs.
- Streaming excluded mode.
- Multi replication focus.
- Native code generation targets.

## 2026 Output Rule

- Do not present verify-current APIs as guaranteed.
- Mark each modern API as `Stable`, `Project-Enabled`, `Verify-Current`, or `Fallback-Required`.
- Provide stable fallback when code depends on a verify-current API.
- Ask before using modern-only paths in production code.
- For audits, flag every modern API assumption.

## Missed Docs

- For mesh physics and rendering, read `references/topics/MeshFidelityLOD.md`.
- For community standards, youth access, chat, ads, UGC, and policy-sensitive systems, read `references/topics/CommunitySafety2026.md`.

## Brownfield Upgrade

- Detect legacy UI patterns.
- Detect client-heavy authority.
- Detect raw DataStore calls.
- Detect old Humanoid-heavy controller logic.
- Detect deprecated or risky serialization.
- Upgrade in small steps.
- Preserve existing behavior.
- Avoid forced rewrites.

## Modern Luau

- Prefer typed shared contracts.
- Use `--!strict` for core modules when practical.
- Use nonstrict only for fast iteration or legacy migration.
- Avoid overusing types when it slows delivery.
- Use type aliases for remote payloads.
- Keep runtime validation even when code is typed.
- Do not replace validation with type checking.

## Server Authority

- Server owns outcomes.
- Client sends intent.
- Latency compensation is optional and domain-specific.
- Prediction requires reconciliation.
- Combat, movement, and interaction must keep server validation.
- Never rely on modern authority features without fallback validation.

## Streaming

- Treat StreamingEnabled as production default for large worlds.
- Critical server logic stays server-side.
- Client code must handle missing streamed instances.
- Use explicit loading states.
- Avoid assuming Workspace descendants exist on client.
- Streaming policy choices must be verified against project settings.

## Open Cloud

- Use engine DataStore or ProfileStore-style session locking for normal player profiles.
- Use Open Cloud Data Stores for external tools, admin panels, dashboards, or cross-experience workflows.
- Require Universe ID, API key or OAuth scope, and strict serialization.
- Do not expose Open Cloud credentials to client.
- Treat Open Cloud Configs as LiveOps when user needs live tuning.
- Provide local config fallback.

## MemoryStore

- Use for temporary cross-server coordination.
- Good for matchmaking, queue, auction, cooldown, and server reservation coordination.
- Do not use as permanent player data.
- Handle expiry and failure.

## UI Modernization

- Prefer reusable components.
- Prefer theme tokens over scattered colors.
- Use responsive constraints for phone, tablet, console, and PC.
- Use input-aware UX when project needs gamepad, touch, or keyboard.
- Treat UI Stylesheets and styling transitions as verify-current.
- Provide traditional Roblox UI fallback when uncertain.

## Avatar And Interaction

- Keep legacy Humanoid path as stable fallback.
- Treat AvatarAbilities-style controller libraries as project-enabled or verify-current.
- Keep movement and combat validated server-side.
- Treat DragDetectors as useful for in-world interaction when available.
- Provide custom input fallback when unavailable.

## Audio And Presentation

- Use modern audio objects only when project target supports them.
- Keep audio managers isolated.
- Avoid per-object audio spam.
- Treat acoustic simulation, advanced emitters, and listeners as verify-current.
- Provide basic SoundService fallback.

## Moderation And Social

- Use TextChatService for modern chat flows.
- Treat age group, trusted connection, moments, TTS, STT, and moderation APIs as verify-current.
- Never bypass Roblox safety systems.
- For UGC-like user content, require moderation path.

## Monetization And LiveOps

- Developer Products and Game Passes must be validated server-side.
- Subscriptions or newer monetization features require current API verification.
- Receipt grant must be idempotent.
- LiveOps config must have safe defaults.
- Analytics events must not block gameplay.

## Serialization

- Treat external JSON and Open Cloud payloads as strict contracts.
- Validate number edge cases.
- Avoid assuming `NaN`, `inf`, or special numeric values serialize normally.
- Use explicit encode/decode adapters for external systems.

## Modern API Gate

- Is this API available in the target environment?
- Does project already use it?
- Does user want modern API path?
- Is there a stable fallback?
- Does it affect security?
- Does it affect data integrity?
- Does it affect client compatibility?
- Is the output clear about assumptions?

## Output Rule

- For `!modernapi`, list `Stable`, `Project-Enabled`, `Verify-Current`, and `Fallback`.
- For code, implement stable path unless user asks modern-only.
- For audit, flag modern API assumptions.
- For migration, suggest incremental upgrade.

