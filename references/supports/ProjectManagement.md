# Project Management Support
> Roblox Skill Pluz support reference for repo layout, team workflow, toolchain decisions, and release flow.
> Use this file only when project management support is explicitly active.

---

## 1. Purpose

Project Management support helps choose the smallest workflow that stays maintainable as the project grows.
It is for structure, sync, packaging, linting, collaboration, review, and release decisions.

Use it when the request involves:
- project structure
- filesystem to Studio sync
- package setup
- repo organization
- team workflow
- build or deploy flow
- linting or formatting
- release or review process

Do not apply it by default.
Do not mention it unless the request needs it.

---

## 2. Supported Scope

| Area | Use For |
|---|---|
| Rojo | filesystem sync |
| Wally | package management |
| Aftman | tool version pinning |
| Stylua | formatting |
| Selene | linting |
| Git | collaboration and history |
| CI/CD | automation |
| Lune | scripted project tasks |
| Roblox Studio workflow | Studio-first editing |
| folder layout | repo organization |
| branch flow | team change control |
| release flow | shipping discipline |
| review flow | change validation |
| task breakdown | work planning |

---

## 3. Activation Rules

- Activate only when the user asks for project management support.
- Keep support separate from gameplay logic.
- Keep optional tooling separate from core implementation.
- Prefer direct commands and file tree when they reduce ambiguity.
- Show the smallest usable setup first.

---

## 4. Decision Principles

1. Start from the current project shape.
2. Keep Studio, filesystem, and package sync aligned.
3. Use one source of truth for code, config, and versioning.
4. Add tools only when they remove real friction.
5. Skip heavy tooling when the project is small or one-off.
6. Standardize repeated workflows.
7. Keep manual workflows manual when they are rare.
8. Add guardrails early when the project is growing fast.
9. Document the minimum path when other people will reuse it.
10. Add review and validation steps when release risk is high.

---

## 5. Tool Selection Rules

### Rojo
Use when filesystem sync matters.

### Wally
Use when package dependencies matter.

### Aftman
Use when tool versions must stay pinned.

### Stylua
Use when formatting consistency matters.

### Selene
Use when linting and static checks matter.

### Git
Use when collaboration, review, or history matters.

### CI/CD
Use when automation reduces release risk.

### Lune
Use when scripted project tasks need repeatability.

### Roblox Studio workflow
Use when the project is Studio-first and external tooling would add noise.

---

## 6. Workflow Rules

| Situation | Rule |
|---|---|
| Small isolated script | avoid heavy project setup |
| Team project | define clear folder and branch flow |
| Package-driven project | pin versions and document install flow |
| Studio sync needed | keep Rojo explicit |
| Build repeated often | automate it |
| One-time task | keep it manual |
| Release-sensitive change | add review and validation |

---

## 7. Output Rules

- State active support first.
- Show only the tools that matter.
- Show the shortest usable workflow.
- Include file tree only when it changes implementation shape.
- Include commands only when they reduce setup ambiguity.
- Keep the answer tied to the request, not to habit.
- Never mix project management policy into script logic.

---

## 8. Required Output

- `Skill Support: Project Management`
- required files only
- optional tooling separated
- no gameplay logic mixed with support rules
