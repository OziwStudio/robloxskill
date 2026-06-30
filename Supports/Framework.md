# Framework Support

## Activation

- Load this file only when user activates Framework support.
- Do not apply these rules by default.
- Do not recommend frameworks unless they solve the request.

## Scope

- Knit
- ProfileStore
- Fusion
- React-Roblox
- Matter ECS
- Flamework
- RbxUtil
- Promise
- Signal
- Component systems
- Dependency injection

## Use When

- User asks for a framework.
- User asks for Knit, ProfileStore, Fusion, React-Roblox, Matter, or Flamework.
- User asks for scalable service/controller architecture.
- User asks for large team architecture.
- User says `Skill Support: Framework`.

## Rules

- Choose a framework only when it reduces real complexity.
- Keep simple systems framework-free.
- Justify framework choice in one short line.
- Avoid framework lock-in when modules are enough.
- Keep lifecycle deterministic.
- Keep dependencies explicit.
- Keep services server-authoritative.
- Keep controllers client-presentational.
- Prefer ProfileStore for production persistence when user enables framework support or asks for production data persistence.
- Do not mix multiple frameworks unless user requests it.

## Framework Choice

- Use Knit for service/controller convention.
- Use ProfileStore for robust profile persistence.
- Use Fusion for reactive UI.
- Use React-Roblox for component UI at scale.
- Use Matter ECS for many entity-like objects.
- Use Flamework for TypeScript-style architecture.
- Use custom modules when framework overhead is not justified.

## Output

- State active support: `Skill Support: Framework`.
- Show framework dependency in file tree.
- Show framework lifecycle clearly.
- Keep fallback module-only option when useful.
