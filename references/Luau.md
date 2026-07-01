# Luau Reference

## Overview
Use this reference for Luau syntax, typing, Roblox-specific patterns, module design, OOP, async control, performance, and common pitfalls.

## Core Rules
- Declare variables with `local`.
- Keep globals out of generated code.
- Use explicit names.
- Keep side effects visible.
- Handle `nil` explicitly.
- Prefer typed public APIs.
- Use `--!strict` for core modules when practical.
- Keep runtime validation near type boundaries.
- Avoid circular require.
- Avoid magic values.
- Avoid deep nesting.

## When To Use
- General Luau syntax questions.
- Code generation.
- Type annotations and generics.
- Roblox API patterns.
- Module design and refactor safety.
- Async and concurrency logic.
- Performance tuning.
- Debugging sharp edges.

## Types
- Use annotations on public signatures.
- Narrow with `typeof()` and `:IsA()`.
- Export shared types from modules.
- Prefer explicit result types for fallible work.
- Use union types for real branching states.
- Use optional types for nullable data.
- Keep public contracts narrow.
- Use tables when related return values belong together.
- Keep shared data shapes explicit.

```luau
--!strict

export type Result<T> = {
	Ok: boolean,
	Value: T?,
	Error: string?,
}

local function ok<T>(value: T): Result<T>
	return { Ok = true, Value = value, Error = nil }
end

local function fail<T>(message: string): Result<T>
	return { Ok = false, Value = nil, Error = message }
end

local function findPlayer(name: string): Player?
	for _, player in game:GetService("Players"):GetPlayers() do
		if player.Name == name then
			return player
		end
	end
	return nil
end
```

## Lua Fundamentals
- Use `local` everywhere.
- Treat tables as the core data type.
- Use metatables for object-like patterns.
- Prefer one return table over multiple returns when clarity matters.
- Handle nil explicitly when absence matters.
- Use sentinels or `rawget` when missing and nil must be distinguished.

```luau
local M = {}

function M.new()
	return setmetatable({}, { __index = M })
end
```

## Functions
- Use named local functions for reusable logic.
- Use closures when state capture is intentional.
- Use variadic functions only when the call shape is truly open.
- Type both parameters and return values on public functions.

```luau
local function add(a: number, b: number): number
	return a + b
end

local function logMessage(prefix: string, ...: any)
	local parts = { ... }
	print(prefix, table.concat(parts, ", "))
end
```

## Tables
- Tables are array, dictionary, object, and namespace.
- Avoid mixed array-dictionary layouts unless necessary.
- Use `table.clone()` for shallow copy.
- Use `table.freeze()` for config.
- Use `table.concat()` instead of repeated `..` in loops.

## Control Flow
- Use `for`, `while`, and `repeat` with clear bounds.
- Use `continue` for early skip.
- Use `break` for exit.
- Prefer generalized iteration when readable.
- Avoid deep nesting when guard clauses are enough.

```luau
if not player then
	return
end

if not character then
	return
end
```

## Strings
- Use string interpolation for readable composition.
- Remember Lua patterns are not regex.
- Escape pattern characters carefully.

## Roblox Patterns
- Use `game:GetService()` at the top.
- Set `Parent` last.
- Disconnect connections when done.
- Use `task.wait()`, `task.spawn()`, `task.delay()`, `task.defer()`.
- Validate every client request on the server.
- Read before write when touching instance trees.
- Prefer events over polling.

```luau
local Players = game:GetService("Players")

local connection = Players.PlayerAdded:Connect(function(player: Player)
	print(player.Name)
end)

connection:Disconnect()
```

## Module Shape
Keep reusable modules cheap at the top level.

- constants
- dependencies
- private state
- helper functions
- public API
- initialization
- cleanup
- setup/init

## Module and Package Design
- Return a table from modules.
- Use `require` for module loading.
- Keep lazy loading only for expensive modules.
- Version shared APIs when contracts matter.
- Document breaking changes when the API changes.

## OOP
- Use metatables for class-like modules.
- Keep constructor, setup, runtime, and cleanup separate.
- Prefer composition over inheritance unless inheritance is clearly simpler.
- Use `__index` for method lookup.
- Use colon syntax for methods that need `self`.

```luau
local Weapon = {}
Weapon.__index = Weapon

function Weapon.new(name: string, damage: number)
	local self = setmetatable({}, Weapon)
	self.Name = name
	self.Damage = damage
	return self
end
```

## Error Handling
- Use `pcall` and `xpcall` around fallible work.
- Return `nil, error_message` when failure is expected.
- Log errors instead of swallowing them.
- Keep fallbacks explicit.

```luau
local ok, result = pcall(function()
	return workspace:FindFirstChild("Missing")
end)
```

## Async
- Use `task` library, not deprecated globals.
- Use promises or coroutines only when they improve structure.
- Keep cancellation and failure paths explicit.
- Keep yield points obvious.
- Do not yield inside top-level initialization unless required.
- Use coroutines for pause/resume style control only when it is clearer than callbacks.
- Keep async boundaries obvious.

## Common Idioms
- Use `table.insert`, `table.remove`, `table.find`, `table.sort`, `table.concat`.
- Use `FindFirstChild`, `WaitForChild`, `GetChildren`, `GetDescendants`.
- Use CollectionService for tag-based lookup.
- Use `pcall` around fallible Roblox services.
- Use `string.format` or interpolation instead of repeated concatenation.
- Use `task.wait`, `task.spawn`, `task.delay`, and `task.defer` instead of deprecated globals.
- Use `typeof` for Roblox-aware type checks.
- Set `Parent` last after property setup.
- Disconnect every connection you no longer need.

```luau
local success, result = pcall(function()
	return game:GetService("DataStoreService"):GetDataStore("X"):GetAsync("key")
end)
```

## Remote Rules
- Send actions, not trusted state.
- Validate type, ownership, range, and timing on the server.
- Keep request and response payloads small.
- Mirror UI state from server state only.
- Treat remote calls as public input.
- Keep validation at the boundary.

```luau
remote.OnServerEvent:Connect(function(player: Player, action: unknown)
	if typeof(action) ~= "string" then
		return
	end
end)
```

## Sharp Edges
- Luau arrays are 1-based.
- `#` is unreliable on sparse tables.
- `nil` removes a key.
- Tables are reference types.
- Only `nil` and `false` are falsy.
- Equality does not coerce types.
- Loop capture rules differ by loop kind.
- Empty tables are truthy.
- Numeric index 1 is the first item, not 0.
- Sparse arrays make length checks unreliable.
- Tables passed into functions are shared references.
- Empty string and zero are truthy.
- `table.remove` shifts elements; setting nil creates gaps.
- `wait`, `spawn`, and `delay` are deprecated.
- `WaitForChild` can yield forever without a timeout.
- `require` can yield during module load.
- `String` patterns are not regex.
- `BindToClose` is needed for shutdown save safety.
- `RemoteEvent` payloads are attacker-controlled.
- Client state is readable and mutable.

## Sharp Edge Details

### 1-Based Indexing
Arrays start at 1.

### Nil Gaps
Do not use `#` on tables with holes.

### Reference Semantics
Aliased tables change together.

### Falsy Rules
Only `nil` and `false` are false.

### Equality
Luau does not coerce types in comparisons.

### Loop Capture
Loop variables can behave differently depending on loop form.

### Infinite Yield
Always prefer `WaitForChild(name, timeout)` when a missing child is possible.

### Module Require Yield
Keep top-level module work cheap and non-yielding.

### Pattern Syntax
Lua patterns are simpler than regex and must be used carefully.

### Shutdown Safety
Use `BindToClose` for save completion and cleanup on server shutdown.

### Client Trust
Client-side currency, inventory, and reward state are unsafe.

### Event Leaks
Disconnect every `Connect` result and destroy temporary instances.

### Remote Flooding
Avoid frame-by-frame remotes and batch safe state updates.

### Public API Rule
- Keep the public API narrow.
- Keep implementation details private.
- Expose only what callers need.

### Versioning Rule
- Version shared module APIs when they are reused.
- Note breaking changes in a changelog or reference file.
- Keep compatibility in mind for shared helpers.

### Composition Rule
- Prefer composition over deep inheritance.
- Keep responsibilities separated into small modules.
- Avoid clever abstractions that hide ownership.

### Sentinel Rule
- Use sentinel values when missing and nil need to be distinct.
- Use `rawget` when table lookup must bypass metamethod logic.

### Returns Rule
- Prefer a table when multiple outputs are closely related.
- Use multiple returns only when the call site stays clear.

## Anti-Patterns
- Global state abuse.
- Massive modules.
- Hidden side effects.
- Silent errors.
- Premature abstraction.
- Untyped shared contracts.
- Blind client trust.
- Polling where events fit better.
- String building in hot loops.
- Overusing inheritance.
- Ignoring cleanup.
- Using nil to represent array holes.
- Using `#` on sparse tables.
- Using direct string concatenation in large loops.
- Assuming type coercion in comparisons.
- Capturing the wrong variable in closures.
- Yielding in top-level init without a clear reason.
- Trusting client currency or reward values.
- Trusting `ProcessReceipt` without deduplication.
- Ignoring `BindToClose`.
- Flooding remotes every frame.
- Leaving `WaitForChild` infinite in critical paths.

## Review Gate
- Is naming clear?
- Is API predictable?
- Is ownership obvious?
- Is cleanup owned?
- Is error handling explicit?
- Is complexity justified?

## Output Focus
- For code, provide a clean module shape.
- For review, report maintainability risk.
- For bugfix, isolate the cause before rewriting.
- Include a module skeleton when useful.
- Prefer compact examples over long tutorials.
- Keep generated code ready to paste.
- Keep reusable APIs small and versioned.
- Keep returns explicit when failure is possible.
- Keep setup and cleanup visible.

## Specialist Habit
- Name the service boundary when code touches Roblox services.
- Name the state owner and cleanup path when code stores state.
- Explain why any yield is safe when code yields.
- If multiple values are needed, prefer a table when clarity matters.
- If code is reusable, keep the public API small and versioned.
