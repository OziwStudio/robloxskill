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
- If the request is small, keep the framework footprint small.
- Prefer architecture that fits the project instead of architecture that looks impressive.
- Keep the foundation useful for years, not only for the current task.
- Treat architecture as a long-term engineering asset.

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
- 9. Keep the smallest workable path.
- 10. Recheck whether the framework is still justified after the design is clear.

## Selection Rules
- Choose the smallest framework set that solves the real problem.
- Choose a framework only when it reduces complexity.
- Choose no framework when a module-only path is cleaner.
- Choose one major framework style at a time unless the project already mixes them.
- Choose tooling only when the project can support it.
- Choose architecture that the team can maintain.
- Choose patterns that improve developer workflow, not only code shape.

## Do Not Use Framework When
- The system is one isolated Script or LocalScript.
- There is no shared lifecycle problem.
- There is no team or scale benefit.
- Framework setup is larger than the feature.
- User asks for simple copy-paste code.
- Existing project style is framework-free and the feature can stay local.
- The lifecycle can stay inside one script plus a helper module.
- The project does not benefit from a shared service/controller pattern.
- The project will be easier to maintain with plain modules and a clear folder tree.

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
- Keep framework startup deterministic.
- Keep dependency ownership obvious.
- Keep cleanup owned by the module that created the resource.
- Keep project structure readable by the next developer.
- Keep system responsibilities explicit across the whole project.
- Keep future extension paths visible.

## Framework Output Contract
- Show why the framework is needed.
- Show the fallback if the framework is removed.
- Show the lifecycle order.
- Show the authority boundary.
- Show the file tree.
- Show the cleanup path.
- Show the risk of over-abstraction.
- Show how the structure scales with more systems and more developers.

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
- Keep remotes small and typed.
- Keep server services authoritative.
- Keep controllers thin.
- Keep shared modules pure where possible.
- Keep startup predictable across multiple systems.
- Keep feature boundaries visible in the folder tree.

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
- Keep migration steps explicit.
- Keep schema versioning in code.
- Keep load failure behavior defined.
- Keep recovery behavior defined.
- Keep long-lived profiles safe under retries and teleports.

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
- Keep UI-only state out of gameplay state.
- Keep cleanup tied to mount/unmount.
- Keep reactive UI small enough to reason about.
- Keep component ownership clear.

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
- Keep server data external to components.
- Keep store scope limited to UI concerns.
- Keep design-system scale realistic for the team.
- Keep UI composition maintainable over time.

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
- Keep system scheduling predictable.
- Keep components data-only where practical.
- Keep ECS only when entity scale justifies it.
- Keep debugging and iteration realistic for the team.

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
- Do not introduce TS/Flamework casually into Luau-only work.
- Keep TS adoption aligned with existing repo style.

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
- Use focused utilities, not framework lock-in.
- Keep utility usage narrow and explicit.

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
- Use Promise for async composition only when it actually simplifies the flow.
- Keep cancellation and retry paths obvious to the reader.

## Signal Scope

### Use When
- Same-runtime modules need decoupled events.
- Service wants internal event API.

### Technical Rules
- Do not use Signal across client/server.
- Disconnect listeners on cleanup.
- Do not fire unbounded high-frequency signals.
- Prefer direct function calls when coupling is acceptable.
- Keep signal ownership explicit.
- Keep same-runtime decoupling simple.

## Component System Scope

### Use When
- World objects use tags.
- Many instances share behavior.
- CollectionService can attach/detach logic cleanly.

### Technical Rules
- Tag only valid instance classes.
- Keep component lifecycle readable.
- Keep tag rules consistent across the project.
- Attach on tag added.
- Cleanup on tag removed and instance destroyed.
- Keep per-instance state isolated.
- Do not store player data on tagged client objects.
- Use tags only for valid instance classes.

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
- Keep constructor side effects light.
- Use adapters or interfaces where helpful.
- Keep tests and mocks straightforward.
- Keep injected dependencies easy to trace.
- Avoid service locator style indirection.

## Principal Systems Architecture
- Think at the system level, not the script level.
- Evaluate how every service, controller, module, dependency, and communication pattern affects the whole project over time.
- Design boundaries that remain understandable under growth.
- Future-proof the project by keeping ownership, lifecycle, and interfaces explicit.

## Core Architecture Philosophy
- Architecture exists to keep the project maintainable, scalable, understandable, and adaptable.
- Architecture should improve developer experience, reliability, security, performance, and team collaboration.
- Avoid short-term fixes that create long-term technical debt.
- Reduce duplication before inventing abstraction.

## Framework Engineering
- Design the foundation that other systems depend on.
- Build reusable infrastructure.
- Standardize development patterns.
- Improve developer productivity.
- Support large development teams.
- Build systems that survive multi-year development.

## System Design Standards
- Every major system needs clear ownership.
- Every major system needs clear responsibilities.
- Every major system needs defined interfaces.
- Every major system needs predictable behavior.
- Every major system needs minimal coupling.
- Avoid hidden dependencies.

## Scalability Review
- Can it support more systems?
- Can it support more developers?
- Can it support more features?
- Can it support more content?
- Can it support more players?

## Maintainability Review
- Can another engineer understand it?
- Can another engineer modify it safely?
- Can another engineer add features confidently?
- Can responsibilities be traced quickly?

## Reliability Review
- Check failure points.
- Check dependency risks.
- Check data consistency.
- Check service ownership.

## Extensibility Review
- Can future systems be added without major rewrites?
- Can the architecture grow without collapsing into debt?

## Project Standards
- Use modern Roblox development tooling when it adds value.
- Respect source control and team workflow.
- Keep framework conventions consistent.
- Keep bootstrapping deterministic.
- Keep lifecycle order visible.
- Keep project structure scalable.

## Team Scalability
- Design for multiple developers.
- Design for shared standards.
- Design for predictable reviews.
- Design for long-term collaboration.
- Keep responsibilities easy to split.

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
- Is the framework choice smaller than the problem?
- Is the startup order deterministic?
- Is the cleanup path explicit?
- Are dependencies easy to trace?
- Is the team likely to maintain it successfully?

## Framework Engineering
- Design the foundation that other systems depend on.
- Keep architecture useful across multi-year projects.
- Reduce duplication before adding abstraction.
- Solve real problems, not imagined ones.
- Keep architecture clear enough for the next developer.
- Prefer consistency over novelty.
- Keep framework complexity smaller than the problem it solves.
- Prefer consistency over scattered ad hoc patterns.

## Architecture Rules
- Project structure must be predictable.
- Module boundaries must be explicit.
- Dependency flow must be easy to trace.
- Lifecycle management must be deterministic.
- Boot order must not rely on race conditions.
- Startup failures must be recoverable.
- Keep project organization readable at a glance.
- Keep module ownership obvious.
- Keep bootstrap order documented.

## Service Architecture
- Services own clear responsibilities.
- Avoid overlap between services.
- Keep services loosely coupled.
- Avoid God services.
- Use services for gameplay, data, economy, and networking boundaries when needed.
- Keep responsibilities domain-specific.
- Keep services loosely coupled.
- Keep service APIs explicit.

## Controller Architecture
- Controllers manage user-facing behavior.
- Controllers own input and presentation.
- Controllers should not hold business logic.
- Keep UI controllers small and domain-named.
- Keep controllers thin and presentational.
- Keep input ownership separate from state ownership.

## Dependency Rules
- Keep dependencies explicit.
- Keep dependency resolution manageable.
- Avoid hidden global access.
- Keep constructor injection lightweight.
- Keep testability in mind from the start.
- Avoid hidden service-locator style access.
- Prefer explicit wiring over magic registration when possible.

## Module Rules
- Each module needs a clearly defined purpose.
- Avoid multi-purpose modules.
- Keep module ownership obvious.
- Keep reusable modules isolated.
- Avoid circular dependency risks.
- Keep public APIs small.
- Keep reusable modules single-purpose.

## Event Rules
- Signals, bindables, event buses, and messaging need ownership.
- Avoid event chaos.
- Keep event flow debuggable.
- Use events where decoupling helps.
- Use direct calls where coupling is acceptable.
- Keep event sources and listeners easy to trace.
- Avoid event storms.

## Bootstrapping Rules
- Startup sequence must be deterministic.
- Dependency loading must be predictable.
- Failure handling must exist.
- Framework startup must not race itself.
- Initialization order must be documented.
- Keep startup steps deterministic.
- Keep startup errors visible.

## ECS Rules
- Use ECS when entity count or behavior variety is high.
- Components store data.
- Systems mutate through a defined flow.
- Keep scheduling predictable.
- Keep ids stable when persistence matters.
- Avoid ECS for simple services.
- Keep entity ids stable.
- Keep data in components and behavior in systems.

## Workflow Rules
- Improve developer productivity.
- Standardize engineering behavior.
- Reduce technical debt.
- Support long-term project growth.
- Keep architecture understandable by the next developer.
- Keep standards consistent across systems.
- Keep the framework useful, not ceremonial.

## Tooling Rules
- Use development tooling only when it adds real leverage.
- Keep tooling aligned with project structure.
- Keep install and sync workflows clear.
- Keep test and lint integration simple.
- Keep tooling optional where the project is small.
- Keep generated project files aligned with the repo tree.

## Output Focus
- Show architecture choice.
- Show lifecycle.
- Show file tree.
- Show authority boundary.
- Show cleanup plan.
- Show fallback path.

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
