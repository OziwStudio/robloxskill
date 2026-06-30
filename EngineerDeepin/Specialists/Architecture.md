# Architecture Specialist

## Scope

- System boundaries.
- Domain ownership.
- Service design.
- Controller design.
- Dependency management.
- Long-term scalability.

## Activate When

- User asks system design, large feature, folder structure, framework decision, migration, or architecture review.
- Complexity is `Moderate` or `Complex`.
- Multiple systems interact.

## Standards

- Define owner, mutator, consumer.
- Define public interface.
- Define dependency direction.
- Isolate domains.
- Keep services domain-focused.
- Keep controllers presentation-focused.
- Prefer explicit dependencies.
- Avoid architecture drift.

## Domain Rule

- Economy owns currency, shops, and transactions.
- Combat owns damage, skills, and status effects.
- Progression owns XP, levels, and unlocks.
- Inventory owns items, equipment, and storage.
- UI consumes state and sends intent.

## Review Gate

- Is ownership clear?
- Is coupling low?
- Can features grow safely?
- Can teams work independently?
- Can startup order be traced?
- Can data flow be traced?

## Anti-Patterns

- God Service.
- God Controller.
- GameManager blob.
- Circular dependency.
- Hidden dependency.
- Tight coupling.
- Architecture drift.

## Output Focus

- State architecture class.
- Show file tree before code.
- Mention migration risk before breaking changes.
