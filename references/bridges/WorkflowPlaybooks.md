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
2. Read `GameDesign.md`.
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

## Tooling Workflow

1. Use Rojo when filesystem sync is needed.
2. Use Wally when packages are managed externally.
3. Use Selene when linting is part of the project.
4. Use StyLua when formatting is part of the project.
5. Use Lune when Lua runs outside Roblox Studio.
6. Keep tools optional for Studio-only projects.
7. Keep tool usage aligned with the project structure.
8. Keep version pins explicit when teams share the repo.
9. Keep generated paths stable across syncs.

### Rojo Rule
- Keep `.luau` as the main script suffix.
- Map folders and services explicitly.
- Keep `project.json` readable.
- Keep filesystem paths consistent with Studio layout.
- Use `*.server.luau` for server scripts.
- Use `*.client.luau` for client scripts.
- Use plain `*.luau` for modules.
- Use `init.server.luau`, `init.client.luau`, and `init.luau` when folder-as-module flow is needed.
- Keep class mapping explicit in the project file.
- Keep the tree stable so Studio and disk stay aligned.

### Rojo Sync Rule
- Read the tree before writing.
- Sync changes after structural edits.
- Verify the tree after build or serve changes.
- Keep two-way sync expectations realistic.
- Avoid depending on unsupported instance sync behavior.

### Wally Rule
- Use `wally.toml` for package declarations.
- Keep packages versioned.
- Keep dev dependencies separate from runtime dependencies.
- Keep package usage limited to actual need.
- Keep shared packages in the shared realm.
- Keep server-only packages in server realm.
- Keep dev-only packages out of runtime paths.
- Use package versions intentionally.

### Wally Layout Rule
- Map installed packages into the project tree explicitly.
- Keep package use visible in the repo.
- Update the package file before asking the agent to rely on it.
- Avoid hidden package assumptions.

### Common Tool Commands
- `rojo init`
- `rojo serve`
- `rojo build`
- `wally install`
- `wally search`
- `wally login`
- `wally publish`

### Lint and Format Rule
- Run linter before publish.
- Run formatter before review when possible.
- Keep formatting deterministic.
- Keep style consistent across the repo.

### Toolchain Rule
- Use Aftman when pinning tool versions matters.
- Keep install commands documented.
- Keep generated project files committed when useful.
- Keep team onboarding simple.
- Keep tool versions pinned in shared projects.
- Keep install order repeatable.
- Keep tool setup low-friction for new contributors.

### Realm Rule
- Use `shared` for server and client.
- Use `server` for server-only packages.
- Use `dev` for development-only packages.
- Keep realm placement consistent with runtime access.

## Studio Mode Workflow

1. Detect Studio mode.
2. Use Full mode when broad MCP exists.
3. Use Standard mode when only limited Studio tools exist.
4. Use Offline mode when no Studio tools exist.
5. Read before write.
6. Verify after modify.

## Tooling Detection
- Check whether the project uses Rojo before assuming filesystem sync.
- Check whether the project uses Wally before assuming package install.
- Check whether the project uses TestEZ before assuming test runner setup.
- Check whether the project uses Lune before assuming external execution.
- Do not force tooling on Studio-only workflows.
- Read project files before assuming tool support.
- Keep tool adoption opt-in for small teams.

## Project Mapping
- Keep script class mapping explicit.
- Keep service folders obvious.
- Keep shared modules in shared space.
- Keep server-only code out of client space.
- Keep UI code in client paths.
- Keep project.json and repo tree in agreement.
- Keep package folders discoverable.
- Keep build output separate from source when practical.

## Install Workflow
1. Pin the toolchain.
2. Install dependencies.
3. Sync the project.
4. Verify the tree.
5. Run tests or lint.
6. Publish only after checks pass.
7. Re-check the tree after publish-oriented edits.

## Onboarding Flow
1. Clone repo.
2. Install toolchain.
3. Install packages.
4. Sync the project.
5. Run tests.
6. Open Studio.
7. Verify path mapping.

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
