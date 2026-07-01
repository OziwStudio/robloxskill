# Directory
> Roblox file system architecture, organization standards, and project structure blueprint.
> Use this file as the single truth for folder taxonomy and asset placement.

## 1. Purpose

The directory structure is the physical backbone of a Roblox project.
A messy directory leads to lost assets, circular dependencies, and unmaintainable code.

Use this structure to:
- enable rapid team navigation
- separate code, assets, and maps
- prevent spaghetti project structure
- support automated builds and sync

Do not use the directory as a dumping ground for test scripts or unoptimized assets.

## 2. Core Principles

1. Separate logic, shared data, and world objects.
2. Give each container one primary purpose.
3. Prefer flat hierarchies for active folders.
4. Use deeper nesting only for meaningful grouping.
5. Keep names consistent and descriptive.
6. Every asset should have a clear home.

## 3. Standard Root Map

| Container | Purpose | Strategy |
|---|---|---|
| Workspace | live game world, physics, visual maps | keep lightweight |
| ReplicatedStorage | shared types, modules, assets, remote objects | source of truth for client and server |
| ServerScriptService | backend logic, data, game loops | server-exclusive entry points |
| ServerStorage | secure assets, private maps | hidden from client inspection |
| StarterPlayer | local controllers, UI, player setup | client-side execution only |
| SoundService | global sound groups, reverb effects | non-playing audio assets |
| Lighting | environment configuration, Atmosphere, Sky | visual presets |

## 4. ReplicatedStorage Taxonomy

Treat ReplicatedStorage as the shared library.

- `Common/` for shared helpers and libraries
- `Components/` for tag-based logic modules
- `Assets/` for media, decals, meshes, animations
- `Types/` for Luau type definitions
- `Events/` for remotes

Rules:
- never require server-only modules from ReplicatedStorage
- keep shared code pure when possible
- keep names stable and explicit

## 5. ServerScriptService Taxonomy

Use ServerScriptService for orchestration and server ownership.

- `Services/` for singleton server systems
- `Systems/` for game bootstrap or mechanics
- `Modules/` for private server-only logic

Pattern:
- create one main script that requires services
- avoid loose root scripts when a folder boundary makes the flow clearer

## 6. Workspace Organization

Use Workspace for the visible stage.

- `Map/` for the static environment
- `Interactables/` for tagged targets
- `Effects/` for non-interactive visuals
- `Entities/` for spawned dynamic objects

Rules:
- every object in Workspace should have a purpose
- decorative props should still have a clear grouping
- game logic should not live in Workspace

## 7. Naming Conventions

Consistency reduces cognitive load.

- folders: PascalCase
- scripts: PascalCase
- modules: PascalCase
- remotes: `Verb_Subject`
- instances: descriptive names, not `Part1`, `Part2`

## 8. Rojo / Sync Integration

If using external editors:

- `src/` should mirror the Roblox hierarchy
- `.project.json` should map physical paths clearly
- avoid manual file moves while sync is active

## 9. Performance Rules

Directory shape affects lookup cost and project clarity.

- do not over-nest folders
- avoid expensive descendant scans when a registry or direct path will do
- keep StreamingEnabled layouts readable and safe for replication

## 10. Security Rules

Directory structure is part of security.

- use ServerStorage for sensitive assets
- keep server-only logic out of client-accessible places
- do not place server scripts inside client containers
- keep boundary crossings explicit through remotes

## 11. Anti-Patterns

### 11.1 God Script

Problem: everything lives in one script.
Fix: split into services and modules.

### 11.2 Dumping Ground

Problem: UI, data, and map objects are mixed together.
Fix: follow the root map.

### 11.3 Mixed Assets

Problem: scripts and models are stored together without purpose.
Fix: separate code from content.

## 12. Maintenance Workflow

- delete copy or test folders regularly
- split folders that grow too large
- refactor repeated long paths
- rename assets so they describe their role

## 13. Professional Behavior Rules for AI

- respect the current project shape
- avoid hardcoding paths when a shared root makes more sense
- suggest structure changes only when they reduce friction
- prefer readability over folder excess

## 14. One-Touch Rule

If a change requires searching too many folders, the system is too coupled.

- logic should be easy to find
- configuration should be easy to find
- instances should be easy to find

## 15. Final Rule

The directory is the map to the project.
If the map is unreadable, the project becomes harder to build, debug, and extend.

