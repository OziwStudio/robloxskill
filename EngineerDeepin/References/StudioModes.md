# Studio Modes

## Purpose

- This reference adopts MCP/orchestration awareness.
- Use when the user asks to build directly in Studio, debug through console, inspect project, or automate a place.
- If Roblox MCP tools are unavailable, use Offline Mode.

## Mode Detection

| Mode | Detect | Behavior |
|---|---|---|
| `Full` | broad Studio MCP tools exist | inspect, edit, build, test, iterate |
| `Standard` | limited official Studio tools exist | run code, read output, playtest |
| `Offline` | no Studio tools | output copy-paste code and file map |

## Full Mode Rule

- Read project tree first.
- Search scripts before editing.
- Read source before write.
- Create undo waypoint before bulk edit.
- Modify in batches.
- Playtest after changes.
- Read output after playtest.

## Standard Mode Rule

- Use code execution to print tree when no tree tool exists.
- Use console output as feedback.
- Check Studio mode before runtime operations.
- Prefer small verified changes.
- Stop playtest before applying edit-time changes.

## Offline Mode Rule

- Output exact placement.
- Output script type.
- Output file tree first.
- Output copy-paste blocks.
- Include manual setup checklist.
- Include verification checklist.

## Safety Rule

- Never assume tool exists.
- Never bulk delete without confirmation.
- Never clear major services without confirmation.
- Never overwrite script blindly.
- Verify after modify.
- Stop after repeated failure and report findings.

## Debug Loop

1. Capture exact output.
2. Locate source.
3. Classify root cause.
4. Patch smallest area.
5. Playtest or provide manual test.
6. Repeat up to 5 iterations.
7. Report remaining risk.

## Template Rule

- If `Templates/` exists in Roblox Skill Pluz, read matching template.
- If `Templates/` is missing, ask user to copy templates first or continue with built-in genre rules.
