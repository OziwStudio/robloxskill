# Studio Modes

## Purpose
- Use when the user asks to build directly in Studio, debug through console, inspect a place, automate changes, or work through MCP tools.
- If MCP is unavailable, fall back to copy-paste output.

## Mode Detection
| Mode | Detect | Behavior |
|---|---|---|
| `Full` | community Studio MCP tools exist | inspect, edit, build, test, iterate |
| `Standard` | official Studio MCP tools exist only | run code, read output, playtest |
| `Offline` | no Studio tools | output file tree and copy-paste code |

## MCP Detection
- Probe for characteristic tool names on session start or when Studio work is requested.
- Prefer the community server when both are available.
- Use official tools only when no community equivalent exists.
- If no tool is available, switch to Offline Mode immediately.
- Do not guess tool names that are not present.

### Community Server
Detect by tools such as:
- `execute_luau`
- `get_file_tree`
- `grep_scripts`
- `create_build`
- `search_objects`
- `get_instance_properties`
- `get_script_source`
- `get_instance_children`
- `search_by_property`
- `get_class_info`
- `get_selection`
- `import_build`
- `export_build`
- `import_scene`
- `start_playtest`
- `stop_playtest`
- `get_playtest_output`
- `undo`
- `redo`
- `mass_get_property`

### Official Server
Detect by tools such as:
- `run_code`
- `insert_model`
- `get_console_output`
- `start_stop_play`
- `run_script_in_play_mode`
- `get_studio_mode`

## Full Mode Rule
- Read project tree first.
- Search scripts before editing.
- Read source before write.
- Create undo waypoint before bulk edit.
- Modify in batches.
- Playtest after changes.
- Read output after playtest.
- Use Creator Store tools only when needed.
- Use build tools for repeated structure.
- Use search tools before manual traversal.
- Use instance properties before rewriting logic.
- Use selection data when the user points to something in Studio.
- Use class info before relying on an API shape.
- Use history tools to recover safely.
- Use bulk operations only after the pattern is verified.

### Full Mode Workflow
1. Inspect tree.
2. Search target scripts.
3. Read source.
4. Plan smallest edit.
5. Create undo point.
6. Apply change.
7. Playtest.
8. Read output.
9. Repeat if needed.
10. Roll back if the batch breaks the rule.

## Standard Mode Rule
- Use code execution to print tree when no tree tool exists.
- Use console output as feedback.
- Check Studio mode before runtime operations.
- Prefer small verified changes.
- Stop playtest before edit-time changes.
- Use inline traversal scripts for structure inspection.
- Use console logs as the main feedback channel.
- Use `get_studio_mode` before any mode-sensitive action.
- Use `run_script_in_play_mode` only for active play sessions.

### Standard Mode Workflow
1. Check Studio mode.
2. Run inspection code.
3. Read console output.
4. Patch the smallest path.
5. Playtest.
6. Read output.
7. Repeat carefully.
8. Stop playtest before edit-time changes.

## Offline Mode Rule
- Output exact placement.
- Output script type.
- Output file tree first.
- Output copy-paste blocks.
- Include manual setup checklist.
- Include verification checklist.
- For Rojo, include folder structure and mapping.
- Keep output self-contained.
- Prefer file tree before code.
- Include `default.project.json` when a filesystem workflow is requested.
- Label each script block with its target service or folder.

### Offline Output Shape
- file tree
- script headers
- exact placement
- code blocks
- setup checklist
- verification checklist

## Safety Rule
- Never assume a tool exists.
- Never bulk delete without confirmation.
- Never clear major services without confirmation.
- Never overwrite script blindly.
- Verify after modify.
- Stop after repeated failure and report findings.
- Never edit live assumptions without confirmation.
- Never continue broken automation forever.
- Never skip verification after a write.
- Never use destructive history operations without intent.

## Debug Loop
1. Capture exact output.
2. Locate source.
3. Classify root cause.
4. Patch smallest area.
5. Playtest or provide manual test.
6. Repeat up to 5 iterations.
7. Report remaining risk.
8. Stop and escalate if the same failure repeats.

## Orchestration Pattern
Use this when the user wants the agent to build or repair a place end-to-end.

1. Scaffold
2. Generate core systems
3. Wire remotes
4. Insert or place assets
5. Test
6. Fix
7. Re-test

## Project Exploration
- Start from the file tree.
- Read service layout before module layout.
- Trace dependencies before changing shared code.
- Use search before opening many files.
- Prefer direct evidence from source over memory.
- Read the active selection when the user references a specific object.
- Use property reads before source edits when configuration is the issue.
- Use targeted queries instead of full-tree scans when possible.

## Assets Rule
- Search creator assets only when they are needed.
- Check metadata before inserting.
- Preview when visual fit matters.
- Insert only after verifying the asset is suitable.
- Use thumbnails for quick filtering.

## Safety Flow
- Check current mode.
- Check tool availability.
- Confirm destructive scope.
- Use undo points.
- Verify result.
- Stop on repeated failure.

## Standard Mode Adaptation
- Without tree tools, print structure from code.
- Without grep tools, search by script traversal and console output.
- Without property search, inspect instances programmatically.
- Without editor tree access, keep edits smaller and more local.

## Autonomous Build
1. Scaffold structure.
2. Generate core systems.
3. Insert assets.
4. Test.
5. Fix and iterate.
6. Re-run output checks.
7. Stop at a stable result.

## Read Before Write
- Read the file that owns the behavior.
- Read dependent files before editing shared logic.
- Read related references before generating code.

## Verify After Modify
- Run the smallest possible validation.
- Read console output.
- Check the affected structure.
- Confirm no new errors appeared.

## Bulk Modification
- Read affected instances first.
- Change in small batches.
- Verify after each batch.
- Keep rollback path ready.
- Use batch changes only when the pattern is stable.
- Stop batch work if one change breaks the rule set.

## Anti-Patterns
- Running code before checking mode.
- Editing blind without reading source.
- Bulk changes without undo points.
- Assuming a missing tool exists.
- Ignoring output after playtest.
- Blind script overwrite.
- Overusing destructive operations.

## Template Rule
- If `Templates/` exists in Roblox Skill Pluz, read matching template.
- If `Templates/` is missing, ask the user to copy templates first or continue with built-in genre rules.
