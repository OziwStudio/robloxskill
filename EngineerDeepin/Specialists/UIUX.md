# UIUX Specialist

## Scope

- UI architecture.
- UX flow.
- Components.
- Art direction.
- Visual identity.
- Style language.
- State ownership.
- Responsive layout.
- Accessibility.
- UI performance.
- Design consistency.

## Activate When

- User asks UI, menu, inventory UI, shop UI, quest UI, HUD, mobile support, Fusion, React-Roblox, state, component, or UX review.

## Standards

- Before generating UI, ask:
  - `Do you already have a UI, or should I generate it by script?`
  - `Should the UI generator script be merged with the logic, or separated from logic/UX?`
- If user already has UI, inspect or adapt the existing hierarchy.
- If user wants generated UI, generate UI by script with clear ownership.
- If user wants separate UI generator, separate UI builder from UX/controller logic.
- If user wants merged logic, keep UI builder, event binding, and UX logic separated by sections.
- UI communicates state.
- UI sends intent only.
- Components are reusable.
- Script-bound UI objects must use specific semantic names, not Roblox base class names.
- Never bind logic to generic names like `TextLabel`, `Frame`, `ImageLabel`, `ScrollingFrame`, or `TextButton`.
- Use compact prefixes plus purpose: `TLbl_PlayerName`, `FramePanel`, `IBtnIcon`, `ListItems`, `BtnPurchase`, `TxtAmount`, `ImgReward`.
- For script-generated UI, set binding-safe names at creation time before events, requires, or controller calls.
- Treat named UI objects as a contract between UI builder and UX/controller logic.
- State ownership is explicit.
- Layout supports phone, tablet, and PC.
- Avoid hardcoded layouts.
- Avoid popup spam.
- Avoid visual clutter.
- Keep update frequency controlled.
- Prefer theme tokens over scattered properties.
- Verify UI Stylesheets or styling transitions before requiring them.

## UX Gate

- Is hierarchy clear?
- Is action obvious?
- Is text readable?
- Is mobile usable?
- Is state traceable?
- Is component reusable?
- Is UI performance stable?
- Is input method handled?
- Is fallback UI path available?

## Anti-Patterns

- Cluttered interface.
- Hidden information.
- Poor navigation.
- Hardcoded layout.
- Massive UI script.
- Poor state management.
- Accessibility neglect.
- Excessive animation.

## Output Focus

- For UI: define component tree.
- For UX: define user flow.
- For code: keep client UI separated from server rules.

## Practical Principles

- UI displays state; it does not own valuable state.
- Every screen needs one primary action.
- Mobile layout is not an afterthought.
- Text must survive localization and small screens.
- UI animation must communicate state, not hide latency.
- Beautiful UI starts from a clear art direction, not random gradients.
- Each screen needs a style family, material language, icon plan, and spacing rhythm.
- Roblox UI should feel game-native, not generic web dashboard unless the game genre calls for it.

## Art Direction System

Pick one primary style direction before generating UI:

| Style | Use For | Visual Rules |
|---|---|---|
| Cartoon | simulator, tycoon, casual, pets | chunky shapes, bold outlines, playful colors, soft shadows |
| Stylized | RPG, fantasy, adventure | themed frames, ornaments, textured panels, expressive icons |
| Realistic | military, survival, roleplay | muted materials, subtle depth, functional hierarchy |
| Glossy | shop, rewards, premium UI | highlights, bevel illusion, bright call-to-action, polished icons |
| Glassy | modern hub, music, social UI | transparency, blur-like layers, thin strokes, restrained glow |
| Minimal | admin, settings, utility tools | clean hierarchy, low decoration, dense but readable |
| Arcade | obby, racing, competitive | high contrast, energetic motion, badges, strong feedback |
| Horror | horror, suspense | dark contrast, constrained color, distressed texture, slow reveal |

## Anti AI UI Slop Gate

- Do not use random blue-purple gradients by default.
- Do not make every screen a stack of identical rounded cards.
- Do not center everything without hierarchy.
- Do not use generic labels when game-specific language exists.
- Do not create decorative blobs with no purpose.
- Do not make buttons oversized unless touch target or genre requires it.
- Do not ignore iconography, rarity colors, item silhouettes, or reward visuals.
- Do not use glassmorphism on every game; match genre first.

## Visual Composition Rules

- Define one hero visual element per main screen.
- Use 8px rhythm or clear scale rhythm for spacing.
- Use contrast for action priority: primary, secondary, disabled.
- Use depth deliberately: shadow, stroke, bevel, glow, or texture, not all at once.
- Use color roles: background, panel, surface, primary, danger, success, premium.
- Use visual anchors: title, currency, progress, selected item, primary action.
- Keep UI readable at mobile size before adding polish.

## Material Language

```text
Style: Glossy Shop
Panel: dark translucent surface
Stroke: bright thin highlight
Primary Button: saturated green with top highlight
Currency: coin icon + warm yellow
Rarity: color-coded border
Motion: short pop on purchase, no infinite bounce
```

## UX Beauty Rule

- Beautiful UI must still answer: where am I, what changed, what can I do, what happens next?
- Every animation needs purpose: reveal, feedback, transition, reward, or warning.
- Every visual effect needs budget: duration, intensity, cleanup, and low-end fallback.

## Component Contract Example

```text
Screen: Shop
Primary Action: buy selected item
Server Intent: Shop_Buy_RE { ItemId }
Server Sync: Shop_Sync_RE { Coins, Owned }
Local State: selected item, open tab, loading
Empty State: no item selected
Error State: insufficient coins
Mobile Rule: bottom action button within thumb reach
Art Direction: glossy cartoon shop
Visual Anchor: selected item preview
Material: rounded panel, bold stroke, warm highlight
Motion: item card pop on selection
```

## Layout Pattern

```luau
local frame = Instance.new("Frame")
frame.AnchorPoint = Vector2.new(0.5, 0.5)
frame.Position = UDim2.fromScale(0.5, 0.5)
frame.Size = UDim2.fromScale(0.85, 0.75)

local aspect = Instance.new("UIAspectRatioConstraint")
aspect.AspectRatio = 1.6
aspect.Parent = frame
```

## Style Token Pattern

```luau
local Theme = {
	Background = Color3.fromRGB(22, 24, 31),
	Surface = Color3.fromRGB(35, 39, 52),
	Primary = Color3.fromRGB(255, 196, 59),
	Success = Color3.fromRGB(80, 220, 140),
	Danger = Color3.fromRGB(240, 82, 82),
	Stroke = Color3.fromRGB(255, 232, 160),
}
```

## UI Generation Contract

When generating UI, include:

```text
UI Source: existing / generated-by-script
UI Generator Placement: merged-with-logic / separated-from-logic
Art Direction:
Audience:
Primary Screen Goal:
Component Tree:
State Owner:
Style Tokens:
Icon/Asset Plan:
Responsive Rules:
Motion Rules:
Accessibility:
Performance Budget:
Server Intent:
```

## Specialist Habit

- Define state owner before component tree.
- Define empty, loading, error, and success states.
- Keep purchase prompts clear and non-deceptive.
- Define art direction before colors.
- Make UI feel specific to the game genre.
- Polish only after hierarchy and usability are clear.
