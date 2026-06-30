# Settings

## Default

- Support with: `None`.
- Memorize: `Read once`.
- Do not learn optional support by default.
- Read core once, then remember important rules.

## Support With

- `None`: default, no optional support loaded.
- `Management Project`: load `Supports/ProjectManagement.md`.
- `Project Management`: same as `Management Project`.
- `Framework`: load `Supports/Framework.md`.

## Support Rules

- Ask user to choose support when installing or adding this skill.
- Do not load support files until user chooses them.
- Keep support choice active only when user requests it.
- Disable support with `!support-off`.

## Memorize

- `Read once`: default mode.
- `Auto Call Skill When Context Full`: reload core when context was compacted or important rules are missing.

## Memorize Rules

- Use `Read once` unless user chooses another mode.
- In `Read once`, do not reread skill files every turn.
- In `Auto Call Skill When Context Full`, reload only `SKILL.md`, `Cores/Rules.md`, `Cores/Commands.md`, `Cores/Settings.md`, and active support files.
- Do not reload inactive support files.
- After reload, keep only important rules in context.

## Generate Mode

- Default mode: practical first.
- Use deeper analysis only when `!deepin`, `!bugfix`, or `!vulnfix` is active.
