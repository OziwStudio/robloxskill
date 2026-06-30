# Settings

## Default

- Engineer: `Basic`.
- Deepin: `Off`.
- Support with: `None`.
- Memorize: `Read once`.

## Deepin Activation

- `Engineer: Deepin` switches to Deepin.
- `!deepin` switches to Deepin for the current task.
- `Deepin: Solo` uses only DeepinEngineer.
- `Deepin: SpecialistName` activates that specialist focus.
- If user does not choose a specialist, choose the smallest needed set.

## Support With

- `None`: do not read support files.
- `Management Project`: read `Supports/ProjectManagement.md`.
- `Framework`: read `Supports/Framework.md`.
- Support is not learned by default.
- Do not mention inactive support unless user asks.

## Memorize

- Default: `Read once, memorize important rules`.
- Optional: `Auto Call Skill When Context Full`.
- On context compaction, reload only core, active engineer, active support, and active reference.

## Reasoning Depth

- Normal generation: working, error-free, light performance first.
- `!deepin`: deeper architecture and risk reasoning.
- `!bugfix`: stronger reasoning for bug root cause.
- `!vulnfix`: strongest reasoning for exploit path and patch.
- `!audit`: stronger reasoning for findings and priority.

## Missing Info

- `Critical`: must ask before work.
- `Degraded`: continue with stated assumptions.
- `Optional`: continue without asking.
