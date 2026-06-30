# Luau Patterns

## Purpose

- Pakai untuk Luau quality, module patterns, lifecycle, cleanup, typing, and testability.
- Pakai `ImplementationPatterns.md` untuk module skeleton, cleanup container, validator, dan state machine.

## Module Shape

- Module body defines functions only.
- Module body must not yield.
- Return one table.
- Use explicit `Init` for setup.
- Use explicit `Start` for runtime connections.
- Use explicit `Destroy` for cleanup.
- Keep service constructor side-effect light.
- Inject services when testing matters.

## Entry Script Shape

```luau
--!strict

local Service = require(script.Parent.Service)

local service = Service.new({
	Debug = true,
})

local ok, err = pcall(function()
	service:Init()
	service:Start()
end)

if not ok then
	warn("[System][Server]Err:" .. tostring(err))
end
```

## Type Rules

- Use typed public contracts.
- Type remote payloads.
- Keep runtime validation.
- Do not rely on types for security.

## Cleanup

- Store every connection.
- Disconnect on cleanup.
- Use Trove/Maid style containers.
- Clean per-player state on `PlayerRemoving`.
- Destroy one-shot sounds and temporary VFX.
- Clear tables that hold player keys.
- Avoid keeping character references after respawn.
- Use `ImplementationPatterns.md#Cleanup Container` when writing reusable cleanup code.

## Testability

- Keep pure logic separate.
- Inject services for tests.
- Mock services at boundaries.
- Avoid `game:GetService()` inside pure functions.

## Gotchas

- Use `task.wait`, `task.spawn`, `task.delay`.
- Use `WaitForChild` timeout.
- Use `table.remove` for arrays.
- Use `%d`, `%w`, `%s` for Luau patterns.
- Avoid yielding in module top-level.
- Avoid circular `require`.
- Avoid `#table` when arrays can have nil gaps.
- Do not use type annotations as security validation.

## Concrete Patterns

- Module service: `ImplementationPatterns.md#Module Shape`.
- Cleanup: `ImplementationPatterns.md#Cleanup Container`.
- Validation: `ImplementationPatterns.md#Payload Validation`.
- State machine: `ImplementationPatterns.md#Round State Machine`.
