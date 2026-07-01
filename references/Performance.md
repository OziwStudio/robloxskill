# Performance

## Purpose
- Use for CPU, memory, rendering, UI, VFX, networking, mobile, gameplay loops, cleanup, and scale audits.
- This file unifies the Basic, Deepin, and specialist performance guidance.

## Core Budget
- Server target: 60Hz.
- Client target: 60fps.
- Frame budget: 16.6ms.
- Heavy work above ~2ms per frame needs attention.
- Luau is single-threaded per VM.
- Low-end mobile is the first failure target.
- Keep network payloads below what the gameplay needs.
- Treat load time as a budget too.
- Optimize for the weakest supported device first.

## Priority
1. Measure the bottleneck class.
2. Fix leaks.
3. Reduce unbounded work.
4. Reduce replication.
5. Reduce rendering load.
6. Keep code maintainable.
7. Keep mobile viable.
8. Keep network use bounded.

## Profiling
- Use MicroProfiler for CPU spikes.
- Use Script Performance for per-script cost.
- Use memory stats for leak checks.
- Profile before optimizing.
- Measure before and after every fix.
- Reproduce the slow path before changing it.
- Use Stats and console output when profiling tools are not enough.
- Use the smallest reproducible scene when debugging performance.
- Measure on the target device class, not only on desktop.

### Profiling Workflow
1. Capture the slowdown.
2. Identify the hottest subsystem.
3. Reproduce on the target device.
4. Change the smallest likely cause.
5. Measure again.
6. Keep the fix if the result is better.
7. Roll back if the fix shifts the bottleneck without helping.

```lua
debug.profilebegin("MySystem")
work()
debug.profileend()
```

## Server Rules
- Avoid excessive Heartbeat.
- Avoid polling when events work.
- Cache services and repeated instance paths.
- Bound table growth.
- Clean per-player state.
- Use task scheduling carefully.
- Do not run visual animation on the server.
- Do not save DataStore per action.
- Do not fire remotes every frame.
- Consolidate recurring frame work into one dispatcher when possible.
- Keep expensive validation out of tight loops.
- Avoid repeated `FindFirstChild` inside Heartbeat.
- Prefer cached references over tree scans.
- Prefer spatial queries over brute force scans.
- Do not create or destroy instances in a hot frame path.
- Keep hot-path validation tiny.

### Heartbeat Manager
```lua
local RunService = game:GetService("RunService")

local HeartbeatManager = {}
HeartbeatManager._callbacks = {}

function HeartbeatManager:Register(id, callback)
	self._callbacks[id] = callback
end

function HeartbeatManager:Unregister(id)
	self._callbacks[id] = nil
end

RunService.Heartbeat:Connect(function(dt)
	for _, callback in self._callbacks do
		callback(dt)
	end
end)
```

### Script Optimization
- Cache `GetService` at the top of each module.
- Use one dispatcher instead of many frame connections.
- Avoid polling when events are enough.
- Avoid repeated hierarchy scans in hot paths.
- Preallocate tables when size is known.
- Build strings in tables, then join once.
- Prefer direct array loops over heavy dictionary work in hot code.
- Use lazy work only when the delay is invisible to players.
- Use rate gates for recurring work that does not need per-frame updates.

## Memory Management
- Disconnect events when done.
- Destroy temporary instances.
- Clear player caches on leave.
- Avoid reference cycles in closures.
- Use `Destroying` when cleanup follows instance lifetime.
- Use `Debris` for timed cleanup.
- Release connections, sounds, and transient objects.
- Avoid keeping dead references in module state.
- Avoid closure capture of large tables.
- Keep lifecycle ownership obvious.
- Drop caches on leave or reset.

### Cleanup Pattern
```lua
local cleaner = {}
local conns = {}

function cleaner.add(conn)
	conns[#conns + 1] = conn
end

function cleaner.clear()
	for _, conn in conns do
		if conn.Connected then
			conn:Disconnect()
		end
	end
	table.clear(conns)
end
```

### Object Pooling
```lua
local pool = {}

local function acquire(template)
	local obj = table.remove(pool)
	if not obj then
		obj = template:Clone()
	end
	obj.Parent = workspace
	return obj
end

local function release(obj)
	obj.Parent = nil
	table.insert(pool, obj)
end
```

### Connection Management
- Store every connection you need to clean up.
- Disconnect on player leave, module shutdown, or state reset.
- Avoid accumulating listeners across rounds or reloads.

### Lazy Loading
- Load distant content late when the player cannot use it yet.
- Keep lazy-load safe for re-entry and reconnect.
- Verify that lazy assets do not break gameplay when missing.

## Network And Gameplay
- Server owns damage, rewards, inventory, rank, currency, and cooldown.
- Client owns camera, input, local animation, visual feedback, and UI.
- Validate distance for touch, pickup, trade, combat, carry, and interact systems.
- Use attributes for simple replicated state.
- Use modules for reusable gameplay logic.
- Use configs for tunable values.
- Batch related remotes.
- Keep payloads compact.
- Compress data shape when stable.
- Use integer enums where possible.
- Use `UnreliableRemoteEvent` only for cosmetic or positional data.
- Use one action remote when the pattern is stable.
- Keep replication frequency lower than the frame rate.
- Never mirror server-owned values as writable client truth.
- Reduce remote traffic before adjusting payload complexity.
- Prefer state snapshots over chatty incremental updates when state changes infrequently.

### Remote Names
- `System_Action_RE` for `RemoteEvent`.
- `System_Action_RF` for `RemoteFunction`.
- `System_Action_BE` for `BindableEvent`.
- Keep remotes under `ReplicatedStorage/SystemName/Remotes`.
- Prefer one typed action remote over many small remotes when safe.

### Remote Validation
- Validate player identity.
- Validate argument types.
- Validate ownership.
- Validate distance when spatial.
- Validate cooldown.
- Validate state, not just type.
- Reject extra data that is not declared.
- Never accept currency, inventory, damage, or rank from client as truth.

### Payload Shape
- Keep payloads flat.
- Use ids instead of long strings when safe.
- Use compact tables over verbose blobs.
- Batch repeated state updates.
- Prefer one intent per message.
- Keep rebroadcast frequency lower than the gameplay tick when possible.

## Rendering
- Reduce part count.
- Reduce shadow use.
- Use `Automatic` or `Performance` on noncritical meshes.
- Use `CanCollide = false`, `CanQuery = false`, `CanTouch = false` for decorative objects.
- Use `BulkMoveTo` for many parts.
- Prefer client-side visuals.
- Use `CollisionFidelity` carefully on every collidable MeshPart.
- Use `Box`, `Hull`, or no collision for decoration.
- Avoid precise mesh collision unless gameplay needs it.
- Use `RenderFidelity` wisely on high-poly assets.
- Use manual LOD for pets, NPCs, cosmetics, and distant dynamic objects.
- LOD must not change server-owned gameplay outcomes.
- Keep visible parts per model and scene under control.
- Enable streaming for large spaces.
- Anchor static parts.
- Prefer MeshPart over UnionOperation when practical.
- Use lower collision fidelity when gameplay allows it.
- Keep draw distance bounded.
- Reduce texture resolution when detail is not visible.
- Use the cheapest visual path that still reads well.
- Keep overdraw and post-processing in check.

### Streaming Rules
- Use `Workspace.StreamingEnabled` for large maps.
- Tune streaming radius for nearby content.
- Keep important models persistent.
- Avoid depending on content that may stream out.
- Lazy-load distant content when the system can tolerate it.
- Avoid assuming far content exists until the load confirms.

### Part Count Targets
- Keep model part count reasonable.
- Keep scene part count controlled.
- Avoid large repeated instance trees when a smaller representation works.
- Prefer merged or simplified shapes for decoration.
- Keep collidable part count smaller than visual part count.

### LOD Rules
- Lower detail for distant dynamic objects.
- Keep gameplay-critical objects authoritative regardless of LOD.
- Use visible mesh detail only where players can inspect it.
- Keep LOD invisible to server logic.

## VFX
- Keep emitters bounded.
- Keep particle textures small.
- Pool frequent VFX.
- Destroy one-shot sounds.
- Disconnect camera shake.
- Restore camera after cutscene.
- Cancel or complete tweens before destroying instances.
- Reduce effects on low-end devices.
- Keep VFX local when they do not affect gameplay.
- Scale down effect counts on weaker devices.
- Avoid long-lived unbounded emitters.
- Simplify particles on mobile.
- Avoid tying expensive VFX to server authority.
- Prefer client-local effects for cosmetics.

## Mobile Rules
- Test on low-end devices.
- Reduce part count aggressively.
- Reduce particle count aggressively.
- Keep UI touch-friendly.
- Reduce draw distance on small screens.
- Keep memory usage lower than desktop.
- Assume mobile is the first device to expose a scaling issue.
- Prefer fewer, clearer UI layers.
- Test touch paths and low-memory paths on mobile.
- Keep animation and effects simpler on low-end devices.
- Keep mobile risks visible in every performance review.

## Best Practices
- Profile before optimizing.
- Optimize hot paths first.
- Use spatial queries over brute force scans.
- Pool frequently spawned objects.
- Lazy-load distant content.
- Keep networking compact.
- Test on low-end devices.
- Treat memory leaks as blockers.
- Use the minimal viable optimization.
- Prefer evidence over intuition.
- Re-benchmark after each fix.

## Scale Checks
- Check player count growth.
- Check long server uptime.
- Check heavy update paths.
- Check replication bursts.
- Check asset load behavior.
- Check more NPCs.
- Check more content.
- Check more servers.
- Check larger worlds.

## NPC Performance
- Keep AI update frequency bounded.
- Keep pathfinding limited.
- Keep detection systems cheap.
- Keep combat logic event-driven when possible.
- Avoid per-frame full NPC scans.
- Avoid updating all NPCs every frame.
- Prefer scalable AI architectures.
- Limit expensive path calculations.

## Large World Optimization
- Review terrain, assets, streaming, and world loading.
- Optimize visibility and asset management.
- Keep replication boundaries clear.
- Keep large-world loading predictable.

## Replication Discipline
- Replicate only what clients must know.
- Keep update frequency bounded.
- Avoid redundant state pushes.
- Prefer stable state snapshots when the value changes rarely.

## Rendering Discipline
- Keep visible scene complexity under control.
- Reduce shadows where they do not help gameplay.
- Simplify materials and textures where players cannot inspect detail.
- Avoid expensive effects stacking in the same view.
- Keep rendering cost visible in audits.

## UI Performance
- Analyze UI complexity.
- Reduce layout calculations.
- Reduce animation overhead.
- Avoid heavy UI hierarchies.
- Avoid unnecessary redraws.

## Animation Performance
- Review animation frequency.
- Review track management.
- Review resource usage.
- Prevent animation leaks.
- Prevent excessive track creation.

## Deep Costs
- `task.spawn` still needs rate control.
- Parallel Luau only helps for heavy work.
- Constraints add physics cost.
- Humanoid is expensive for simple NPCs.
- Complex mesh collision is expensive.
- Streaming changes client availability.
- DataStore yields the calling thread.
- BindableEvent is not ideal for internal module flow.
- Lazy-load can hide bugs if not verified.
- Pooling can leak if cleanup is missing.

## Fast Rules
- Use `task.wait()`, not `wait()`.
- Use numeric loops for arrays.
- Use `PivotTo` for model moves.
- Keep animation and VFX on the client.
- Keep server authority on gameplay state.
- Keep optimization measurable.
- Prefer batched work over per-frame work.
- Prefer cached data over repeated traversal.
- Prefer stable budgets over ad hoc fixes.
- Prefer pooling over repeated instantiation.
- Prefer lazy-loading distant content.
- Prefer measured fixes over theoretical ones.

## Patterns

```lua
local conn
conn = RunService.Heartbeat:Connect(function()
	if done then
		conn:Disconnect()
	end
end)
```

```lua
local acc = 0
RunService.Heartbeat:Connect(function(dt)
	acc += dt
	if acc < 0.1 then
		return
	end
	acc = 0
	updateAll()
end)
```

```lua
local hitBox = model:FindFirstChild("HitBox")
RunService.Heartbeat:Connect(function()
	if hitBox and hitBox.Parent then
		movePart(hitBox)
	end
end)
```

```lua
local scratch = {}
local function getActiveEnemies()
	table.clear(scratch)
	for _, enemy in enemies do
		if enemy.active then
			scratch[#scratch + 1] = enemy
		end
	end
	return scratch
end
```

```lua
local part = workspace:WaitForChild("Part", 5)
if not part then
	return
end
```

```lua
local results = table.create(100)
for i = 1, 100 do
	results[i] = computeValue(i)
end
```

## Anti-Patterns
- Premature optimization.
- Creating and destroying instances in Heartbeat.
- Large uncompressed remote payloads.
- Not testing on mobile.
- Ignoring memory leaks.
- Heavy polling instead of event-driven flow.
- Optimizing before measuring.
- Updating too many NPCs every frame.
- Recreating connections on every state change.
- Re-broadcasting unchanged replicated data.
- Excessive Heartbeat usage.
- Excessive RenderStepped usage.
- Massive instance counts.
- Remote spam.
- Memory leaks.
- Connection leaks.
- Constant polling.
- Redundant calculations.
- Unbounded tables.
- Physics abuse.

## Performance Review Framework
- CPU: how expensive is execution?
- Memory: how expensive is storage?
- Network: how expensive is replication?
- Rendering: how expensive is display?
- Scalability: how does the system behave at scale?

## Performance Audit Format
```text
[performance_strengths]

[performance_issues]

[impact]

[recommendations]

[priority]
```

## Performance Benchmarks
- Measure frame consistency.
- Measure memory stability.
- Measure CPU stability.
- Measure network stability.
- Measure long-session performance.
- Optimize toward measurable metrics.

## Technical Leadership
- Think like a principal performance engineer.
- Think like a scalability engineer.
- Think like a technical director.
- Do not optimize based on assumptions.
- Optimize based on evidence.

## Communication Style
- Be analytical.
- Be technical.
- Be data-driven.
- Be objective.
- Support conclusions with measurable reasoning.

## Final Verification
- CPU usage optimized.
- Memory usage controlled.
- Network traffic efficient.
- Replication optimized.
- Streaming compatibility verified.
- Resource cleanup implemented.
- Scalability validated.
- Profiling performed.
- Performance bottlenecks addressed.
- Long-term stability maintained.

## Final Objective
- Identify bottlenecks.
- Improve efficiency.
- Reduce resource consumption.
- Keep experiences smooth and scalable.
- Follow modern Roblox engineering standards.
- Repeated hierarchy scans in hot loops.
- Blind remote flooding.

## Output Rule
- Report the bottleneck first.
- Name the hot path.
- Give the smallest safe fix.
- Mention tradeoffs briefly.
- For audits, separate CPU, memory, rendering, networking, and gameplay costs.
- Include the target platform if it matters.
- Call out any tradeoff that shifts load elsewhere.

## Audit Focus

- Use this file as the source of truth for performance audit rules.
- Start with profiling data, not assumptions.
- Treat hot-path CPU, memory leaks, replication, rendering, and mobile viability as the main audit buckets.
- Treat repeated work, unbounded loops, and connection leaks as top risks.
- Treat part count, mesh complexity, and unnecessary replication as scale risks.
- Treat cleanup and bounded update rates as mandatory gates.

## Audit Checks

- Check `wait()`/`spawn()`/`delay()` usage.
- Check repeated `GetService` calls in hot paths.
- Check multiple `Heartbeat` or `RenderStepped` connections.
- Check `Instance.new` in loops or hot paths.
- Check undisconnected events and per-player leaks.
- Check remote spam and oversized payloads.
- Check part count, unanchored physics, and mesh complexity.
- Check low-end mobile behavior first.

## Threshold Guide

- Over ~2ms per frame: needs attention.
- Over 10,000 parts in a scene: requires culling or streaming review.
- Over 50,000 parts: critical redesign territory.
- Over 10,000 triangles per MeshPart: flag for review.
- Fire remotes in controlled bursts, not in frame loops.
- Keep hot-path validation tiny.
