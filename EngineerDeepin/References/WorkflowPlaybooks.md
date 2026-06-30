# Workflow Playbooks

## Debug Loop

1. Gather exact error, file, line, and reproduction.
2. Inspect surrounding code.
3. Classify bug: syntax, runtime, logic, security, data, performance.
4. Identify root cause.
5. Patch smallest correct area.
6. Check regression path.
7. Summarize cause, fix, and verification.

## Code Review

1. Scan project shape.
2. Check architecture boundaries.
3. Check server authority.
4. Check data safety.
5. Check remote validation.
6. Check memory cleanup.
7. Check performance hot spots.
8. Report findings first.

## Security Audit

1. List remote surface.
2. List trusted inputs.
3. Find client authority.
4. Find missing validation.
5. Find rate-limit gaps.
6. Find data exposure.
7. Patch exploit path.
8. Re-check abuse cases.

## Performance Audit

1. Find loops, events, and repeated work.
2. Find instance churn.
3. Find replication cost.
4. Find unbounded tables.
5. Find connection leaks.
6. Find mobile risks.
7. Prioritize measurable fixes.

## Monetization Audit

1. Check ProcessReceipt idempotency.
2. Check grant before save risk.
3. Check duplicate rewards.
4. Check client purchase trust.
5. Check developer product mapping.
6. Check failure handling.
7. Check logs and debug output.

## Publish Checklist

1. No insecure remotes.
2. No client-owned rewards.
3. No unsafe receipt path.
4. No obvious save loss path.
5. No connection leaks in main systems.
6. No infinite yielding in startup.
7. No debug spam unless intended.
8. File tree matches architecture.
9. Header, footer, version, comments, debug rule pass.

## New Game Workflow

1. Define core loop.
2. Define player progression.
3. Define economy source and sink.
4. Define persistence.
5. Define multiplayer interactions.
6. Define UI flow.
7. Define monetization only after fairness.
8. Define first playable milestone.

## Complete Game Workflow

1. Detect genre.
2. Read `CompleteGameLayer.md`.
3. Read `TemplateBridge.md`.
4. Read matching template from `Templates/` if present.
5. Define first 60 seconds.
6. Define scaffold.
7. Generate core systems.
8. Add QA plan.
9. Run publish gates.
10. Summarize next content updates.

## Testing Workflow

1. Identify critical paths.
2. Separate pure logic from engine glue.
3. Define unit tests for pure logic.
4. Define integration tests for data flow.
5. Define remote abuse tests.
6. Define manual mobile and multiplayer tests.
7. Define regression checks.

## Studio Mode Workflow

1. Detect Studio mode.
2. Use Full mode when broad MCP exists.
3. Use Standard mode when only limited Studio tools exist.
4. Use Offline mode when no Studio tools exist.
5. Read before write.
6. Verify after modify.

## Report Format

```text
[summary]
[findings]
[root_cause]
[risk]
[fix]
[verification]
[next]
```
