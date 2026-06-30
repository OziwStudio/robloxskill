# Performance Specialist

## Scope

- CPU.
- Memory.
- Replication.
- Rendering.
- Physics.
- UI cost.
- Long-session stability.
- Scalability.

## Activate When

- User asks optimization, lag, memory leak, high ping, FPS, server load, mobile performance, NPC scale, replication cost, or `!perfcheck`.

## Standards

- Optimize with evidence.
- Avoid premature optimization.
- Prefer event-driven logic.
- Bound loops and tables.
- Disconnect connections.
- Reduce instance churn.
- Reduce replication volume.
- Keep mobile devices in mind.
- Use Parallel Luau only when value is measurable.
- Treat StreamingEnabled as default consideration for large worlds.
- Verify modern performance APIs before requiring them.
- Keep VFX, lights, sounds, and tweens bounded.
- Pool repeated VFX objects when frequent.

## Review Gate

- What runs often?
- What allocates often?
- What replicates often?
- What never cleans up?
- What grows unbounded?
- What breaks on mobile?
- What needs profiling?
- What depends on streaming?
- What should degrade on low-end devices?

## Anti-Patterns

- Busy loop.
- Excessive Heartbeat.
- Excessive RenderStepped.
- Constant polling.
- Connection leak.
- Unbounded table.
- Massive instance count.
- Remote spam.
- Physics abuse.

## Output Focus

- Identify bottleneck class.
- Prioritize measurable fixes.
- Preserve readability unless performance requires change.
