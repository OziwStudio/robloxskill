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

## Priority
1. Measure the bottleneck class.
2. Fix leaks.
3. Reduce unbounded work.
4. Reduce replication.
5. Reduce rendering load.
6. Keep code maintainable.

## Profiling
- Use MicroProfiler for CPU spikes.
- Use Script Performance for per-script cost.
- Use memory stats for leak checks.
- Profile before optimizing.

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

## Client Rules
- Mobile is first-class.
- Avoid hardcoded UI.
- Reduce visible part count.
- Use StreamingEnabled for large worlds.
- Degrade VFX on low-end devices.
- Keep UI, camera, and local effects client-side.

## Main Risks
- Heartbeat loops with heavy work.
- Connection leaks.
- Instance polling.
- Table creation in hot loops.
- String concatenation in loops.
- `FindFirstChild` in hot paths.
- `GetDescendants` in hot paths.
- Remote flooding.
- Infinite `WaitForChild`.
- Recursive `require` cycles.

## Safer Patterns
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

## Gameplay And Networking
- Server owns damage, rewards, inventory, rank, currency, and cooldown.
- Client owns camera, input, local animation, visual feedback, and UI.
- Validate distance for touch, pickup, trade, combat, carry, and interact systems.
- Use attributes for simple replicated state.
- Use modules for reusable gameplay logic.
- Use configs for tunable values.
- Batch remote updates.
- Keep payloads compact.
- Use integer enums where possible.
- Use `UnreliableRemoteEvent` only for cosmetic or positional data.

```lua
local lastSent = 0
RunService.Heartbeat:Connect(function()
	local now = os.clock()
	if now - lastSent < 0.05 then
		return
	end
	lastSent = now
	RemoteEvent:FireAllClients(data)
end)
```

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
- Never accept currency, inventory, damage, or rank from client as truth.

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

## VFX
- Keep emitters bounded.
- Keep particle textures small.
- Pool frequent VFX.
- Destroy one-shot sounds.
- Disconnect camera shake.
- Restore camera after cutscene.
- Cancel or complete tweens before destroying instances.
- Reduce effects on low-end devices.

## Architecture
- Cache `GetService` results at module top.
- Prefer one heartbeat dispatcher per system.
- Pool frequently spawned objects.
- Keep module dependencies acyclic.
- Use attributes for scalar instance data.
- Use direct loops over flat tables.

```lua
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
```
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
```

## Deep Costs
- `task.spawn` still needs rate control.
- Parallel Luau only helps for heavy work.
- Constraints add physics cost.
- Humanoid is expensive for simple NPCs.
- Complex mesh collision is expensive.
- Streaming changes client availability.
- DataStore yields the calling thread.
- BindableEvent is not ideal for internal module flow.

## Memory
- Watch LuaHeap growth.
- Clear player caches on leave.
- Disconnect character and player signals.
- Avoid closure capture of large tables.
- Avoid recursive thread spawning.
- Use direct array append for tail adds.
- Use `table.concat` for strings.

## Fast Rules
- Use `task.wait()`, not `wait()`.
- Use numeric loops for arrays.
- Use `PivotTo` for model moves.
- Keep animation and VFX on the client.
- Keep server authority on gameplay state.
- Keep optimization measurable.

## Output Rule
- Report the bottleneck first.
- Name the hot path.
- Give the smallest safe fix.
- Mention tradeoffs briefly.
- For audits, separate CPU, memory, rendering, networking, and gameplay costs.
