# Commands

## Reading

| Command | Rule |
|---|---|
| `!readall` | Read `SKILL.md`, Basic core, and needed Basic references once. |
| `!read-core` | Read `EngineerBasic/Cores`. |
| `!read-settings` | Read `EngineerBasic/Cores/Settings.md`. |
| `!read-rules` | Read `EngineerBasic/Cores/Rules.md`. |
| `!read-commands` | Read `EngineerBasic/Cores/Commands.md`. |
| `!read-model` | Read `EngineerBasic/Cores/AIModel.md`. |
| `!read-output` | Read `EngineerBasic/Cores/ChatOutput.md`. |
| `!read-architecture` | Read `EngineerBasic/References/Architecture.md`. |
| `!read-networking` | Read `EngineerBasic/References/Performance.md` networking section. |
| `!read-security` | Read `EngineerBasic/References/Architecture.md` security section. |
| `!read-data` | Read `EngineerBasic/References/Architecture.md` data section. |
| `!read-luau` | Read `EngineerBasic/References/Architecture.md` Luau patterns section. |
| `!read-systems` | Read `EngineerBasic/References/Architecture.md` systems section. |
| `!read-uiux` | Read `EngineerBasic/References/UIUX.md`. |
| `!read-performance` | Read `EngineerBasic/References/Performance.md`. |
| `!read-gameplay` | Read `EngineerBasic/References/Performance.md` gameplay section. |
| `!read-support-framework` | Read `Supports/Framework.md`. |

## Skill Support

| Command | Rule |
|---|---|
| `!support-list` | Show optional support choices only. |
| `!support-none` | Keep optional support disabled. |
| `!support-management-project` | Activate Management Project support and read `Supports/ProjectManagement.md`. |
| `!support-framework` | Activate Framework support and read `Supports/Framework.md`. |
| `!support-off` | Disable optional support. |

## Memorize

| Command | Rule |
|---|---|
| `!memorize-read-once` | Use default mode: read once and remember important rules. |
| `!memorize-auto-context` | Auto reload core when context was compacted or important rules are missing. |

## Generation

| Command | Rule |
|---|---|
| `!genfull` | Generate full production system with classifier, file tree, then code. |
| `!gensnip` | Generate only the requested snippet and integration notes. |
| `!bugfix` | Focus on bug fixes and breakages. |
| `!vulnfix` | Focus on security vulnerabilities. |
| `!refactor` | Refactor provided code to these rules. |
| `!audit` | Return security, performance, architecture, and bug findings. |
| `!explain` | Explain code or concept clearly. |
| `!diagram` | Output ASCII structure or flow. |
| `!modelpro` | Use concise technical tone. |
| `!modelfun` | Use creative tone while keeping code strict. |
| `!whitelist` | Ask user for the `UserId` to auto-include. |
| `!discuss` | Discuss only; do not write code. |
| `!createcmd` | Add or revise command definitions. |

## Keyword Rules

- `architecture`: read Architecture reference.
- `settings`: read settings only.
- `deepin`: read Deepin only when activated.
- `networking`: read Performance reference networking section.
- `security`: read Architecture reference security section.
- `data`: read Architecture reference data section.
- `luau`: read Architecture reference Luau patterns section.
- `systems`: read Architecture reference systems section.
- `uiux`: read UIUX only.
- `performance`: read Performance reference.
- `gameplay`: read Performance reference gameplay section.
- `project-management`: read Project Management support only when activated.
- `framework`: read Framework support only when activated.
- `bugfix`: focus on bug fixing mode.
- `vulnfix`: focus on security fixing mode.

## Token Rule

- Do not reread skill files every turn.
- Keep the important rules in context after first read.
- Load reference files only when the command or task needs them.
- Load support files only when user activates support.
- Default support is `None`.
- Default memorize is `Read once`.

## Generate Focus

- Default generate goal: make it work, keep it light, avoid overthinking.
- `!bugfix`: solve functional bugs first.
- `!vulnfix`: solve security holes first.
- Deep complexity analysis is secondary unless user asks for it.

## Engine Switch

- `Engineer: Basic` is the only default mode visible here.
- If user asks for Deepin, hand off to `EngineerDeepin/Cores/Commands.md`.
