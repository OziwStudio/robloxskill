# UIUX

## Purpose

- Pakai untuk GUI, HUD, shop, inventory, mobile, gamepad, UI state, and UX review.

## UI Rules

- UI communicates state.
- UI sends intent only.
- Server owns outcome.
- Use reusable components.
- Use responsive layout.
- Support phone, tablet, PC, and gamepad when relevant.
- Avoid popup spam.

## Layout

- Prefer scale and constraints.
- Use `UIScale`, `UISizeConstraint`, `UIAspectRatioConstraint`.
- Use `UIListLayout`, `UIGridLayout`, `UIPadding`.
- Avoid hardcoded pixels for main layout.
- Keep thumb reach in mind.
- Use nested frames for dense layouts.
- Use `AutomaticSize` only with clear bounds.
- Prevent alpha bleeding in icon assets with pixel padding before upload.
- Disable `AutoLocalize` for brand names, fantasy terms, and special names when translation would corrupt meaning.

## Patterns

- Shop: clear value, clear price, no deceptive prompt.
- Health bar: readable, animated but bounded.
- Notification: short, stack-limited, auto-dismiss.
- Dialog: explicit confirm/cancel.
- Inventory: filtering, search, capacity, item details.

## Performance

- Do not create UI every frame.
- Pool repeated UI when needed.
- Disconnect UI events on close/destroy.
- Avoid blocking UI thread with yields.

## Anti-Patterns

- Hardcoded layout.
- Cluttered screens.
- Hidden information.
- Massive LocalScript.
- Client game logic.
- Ignoring mobile and gamepad.
