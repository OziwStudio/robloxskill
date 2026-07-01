# Performance Audit Workflow

This workflow orchestrates the performance audit. The source of truth for rules and thresholds lives in `references/Performance.md`.

---

## Step 1: Project Scan

- Map the project tree.
- Gather profiler data when available.
- Look for hot paths, repeated work, and obvious anti-patterns.
- If access is partial, ask the user for the missing data.

## Step 2: Attach References

- Read `references/Performance.md` for the core performance rules.
- Read `references/bridges/ImplementationPatterns.md` when you need a concrete coding pattern.
- Read `references/topics/StudioModes.md` when MCP or Studio mode matters.

## Step 3: Audit Order

1. CPU and hot-path work.
2. Memory leaks and cleanup.
3. Replication and network traffic.
4. Rendering and part count.
5. Mobile and low-end risk.
6. Scalability and long-session behavior.

## Step 4: Report

- Lead with the bottleneck.
- Name the hot path.
- Give the smallest safe fix.
- Separate CPU, memory, rendering, networking, and gameplay costs.
