# Framework Support

## Activation
- Load this file only when user activates Framework support.
- Do not apply these rules by default.
- Do not recommend frameworks unless they solve the request.
- State active support: `Skill Support: Framework`.
- Keep the operating principle: `POWERFUL BUT TOKEN-EFFICIENT`.

## Purpose
- Use this support for Roblox framework decisions, service/controller architecture, production persistence, reactive UI, ECS, dependency injection, async flow, reusable utility modules, and large-team project structure.
- Framework support must make the solution clearer, safer, or easier to scale.
- If plain ModuleScripts are enough, choose plain ModuleScripts.

## Scope Map
| Scope | Use For | Default Position |
|---|---|---|
| Knit | service/controller convention, remotes abstraction, medium-large systems | recommended when service boundaries matter |
| ProfileStore | production player profiles, session locking, migrations | preferred persistence support |
| Fusion | reactive Roblox UI, smaller component UI, state-driven HUD | use when UI state is complex but React is too heavy |
| React-Roblox | large UI apps, many reusable components, team UI workflow | use when UI scale justifies overhead |
| Matter ECS | many entities, simulation, components, systems | use for entity-heavy gameplay |
| Flamework | TypeScript-style dependency injection and decorators | use only when project already uses TS/Flamework |
| RbxUtil | utility modules: Signal, Trove, Promise, TableUtil, etc. | use as focused dependency, not as architecture |
| Promise | async flows, loading, retries, cancellation boundaries | use when async composition is real |
| Signal | decoupled in-memory events | use inside same runtime only |
| Component Systems | tagged instances, CollectionService behaviors | use for world objects and reusable interactions |
| Dependency Injection | testable services, explicit dependencies | use for complex services and mocks |

## Decision Flow
1. Identify project type: greenfield or brownfield.
2. Identify existing framework.
3. Identify requested framework.
4. Identify risk: data, security, networking, UI scale, simulation scale.
5. Choose the smallest framework set.
6. Define fallback module-only path when useful.
7. State framework choice in one line.
8. Generate code using the chosen lifecycle.

## Do Not Use Framework When
- The system is one isolated Script or LocalScript.
- There is no shared lifecycle problem.
- There is no team or scale benefit.
- Framework setup is larger than the feature.
- User asks for simple copy-paste code.
- Existing project style is framework-free and the feature can stay local.

## Global Rules
- Server services own gameplay outcomes.
- Client controllers own input and presentation.
- Shared modules own types, constants, contracts, and pure helpers.
- Data profiles are mutated only by server services.
- UI frameworks must not own server authority.
- Framework lifecycle must be deterministic.
- Dependencies must be explicit.
- Do not mix multiple major frameworks unless user requests it or project already does.
- Do not hide security behind framework abstractions.
- Keep require paths extension-free.

## Knit Scope

### Use When
- System needs clear server `Service` and client `Controller`.
- Multiple services need lifecycle order.
- Remotes need consistent convention.
- Team needs predictable boundaries.

### Structure
```text
ServerScriptService/
  SystemServer/
    Services/
      CurrencyService.luau
      InventoryService.luau
    Main.server.luau

StarterPlayerScripts/
  SystemClient/
    Controllers/
      ShopController.luau
    Main.local.luau

ReplicatedStorage/
  SystemName/
    Modules/
      Types.luau
      ConfigShared.luau
```

### Technical Rules
- Services expose server-owned methods.
- Controllers call services through declared client APIs.
- Never trust controller data.
- Keep business logic inside services or pure shared modules.
- Avoid service-to-service require cycles.
- Prefer explicit dependencies over hidden global access.

### Output Must Include
- Service list.
- Controller list.
- Shared module list.
- Lifecycle order.
- Remote/client API surface.

## ProfileStore Scope

### Use When
- Player data has value.
- Data needs session locking.
- There are purchases, inventory, progression, economy, or migration.
- User asks for production persistence.

### Technical Rules
- Define `CURRENT_DATA_VERSION`.
- Define default profile template.
- Reconcile missing fields on load.
- Run migrations sequentially.
- Keep receipt idempotency in profile data.
- Save before teleport when server hopping.
- Use Studio/dev prefix for testing.
- Never expose profile object to client.

### Required Output
```text
Profile Key:
Default Schema:
Migration Steps:
Load Failure Behavior:
Save Lifecycle:
Receipt Idempotency:
Studio Test Prefix:
```

## Fusion Scope

### Use When
- UI has reactive state.
- HUD/shop/inventory updates often.
- Project wants Lua-native component UI.
- UI complexity is medium.

### Technical Rules
- Keep authoritative data server-side.
- Client state mirrors server state.
- Store UI state separately from gameplay data.
- Destroy scopes/observers on close.
- Avoid creating reactive objects every frame.
- Keep components small and named by domain.

### Output Must Include
- State sources.
- Component tree.
- Cleanup owner.
- Server sync path.
- Mobile layout constraints.

## React-Roblox Scope

### Use When
- UI has many screens and reusable components.
- Team already knows React patterns.
- UI needs routing, state containers, or design system scale.

### Technical Rules
- Keep server data as external source.
- Use client store only for UI state/cache.
- Avoid putting purchase authority in UI components.
- Keep component props typed.
- Keep mount/unmount cleanup explicit.
- Avoid framework use for one simple HUD.

### Output Must Include
- Component hierarchy.
- Store/state model.
- Remotes read/write boundary.
- Mount location.
- Cleanup and unmount behavior.

## Matter ECS Scope

### Use When
- There are many entities with repeated behavior.
- Simulation is component-driven.
- Systems need predictable iteration.
- Object count or behavior variety is high.

### Technical Rules
- Components store data.
- Systems mutate through defined flow.
- Server owns authoritative gameplay components.
- Client may render predicted or visual-only components.
- Keep entity ids stable when persistence matters.
- Avoid ECS for simple services.

### Output Must Include
- Components.
- Systems.
- Authority owner.
- Replication boundary.
- Cleanup/despawn rule.

## Flamework Scope

### Use When
- Project already uses TypeScript/roblox-ts.
- User explicitly asks for Flamework.
- Team wants decorator-based dependency injection.

### Technical Rules
- Do not introduce Flamework into a Luau-only project casually.
- Respect existing TS folder layout.
- Keep services server-authoritative.
- Keep controllers presentational.
- Keep decorators and dependency injection predictable.

### Output Must Include
- TS/Flamework assumption.
- Service/controller class list.
- Dependency injection map.
- Build/tooling note.

## RbxUtil Scope

### Use When
- Need focused utilities without full framework lock-in.
- Need `Signal`, `Trove`, `Promise`, `TableUtil`, or similar helper.

### Technical Rules
- Import only the utility needed.
- Avoid turning utility modules into architecture.
- Use Trove/Maid-like cleanup for connections and effects.
- Use Signal only inside the same runtime boundary.

## Promise Scope

### Use When
- Multiple async operations must compose.
- Need timeout, retry, cancellation, or load pipelines.

### Technical Rules
- Use Promise for async flow, not for simple one-call work.
- Handle rejection.
- Avoid unbounded chains.
- Cancel work when UI closes or player leaves.
- Do not hide DataStore failure.

## Signal Scope

### Use When
- Same-runtime modules need decoupled events.
- Service wants internal event API.

### Technical Rules
- Do not use Signal across client/server.
- Disconnect listeners on cleanup.
- Do not fire unbounded high-frequency signals.
- Prefer direct function calls when coupling is acceptable.

## Component System Scope

### Use When
- World objects use tags.
- Many instances share behavior.
- CollectionService can attach/detach logic cleanly.

### Technical Rules
- Tag only valid instance classes.
- Attach on tag added.
- Cleanup on tag removed and instance destroyed.
- Keep per-instance state isolated.
- Do not store player data on tagged client objects.

## Dependency Injection Scope

### Use When
- Services need tests.
- Services have replaceable adapters.
- Circular dependencies are a risk.

### Technical Rules
- Pass dependencies explicitly.
- Keep constructor side-effect light.
- Use interfaces/types for adapters.
- Avoid service locator abuse.
- Do not inject raw client-owned authority into server services.

## Framework Architecture Output
When generating a framework-backed system, output:
```text
Skill Support: Framework
Framework Choice:
Reason:
Fallback:
Lifecycle:
File Tree:
Authority:
Data:
Remotes:
UI:
Cleanup:
Risks:
```

## Framework Review Checklist
- Does framework reduce real complexity?
- Is the service boundary clear?
- Is server authority preserved?
- Are dependencies explicit?
- Are remotes validated?
- Is persistence session-safe?
- Is UI only presentation?
- Is cleanup deterministic?
- Is mobile/performance considered?
- Is there a module-only fallback?

## Anti-Patterns
- Adding Knit for one tiny server script.
- Adding React for one static frame.
- Using ECS for a normal shop.
- Using ProfileStore without schema versioning.
- Mixing Knit, Flamework, and custom framework without reason.
- Putting server authority in controllers or UI.
- Hiding remotes inside framework magic without validation.
- Ignoring cleanup because framework has lifecycle hooks.
- Framework migration without reading existing project style.
