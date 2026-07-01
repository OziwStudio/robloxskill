# Monetization Audit Workflow

This workflow orchestrates the monetization audit. The source of truth for rules and technical gates lives in `references/topics/MonetizationAndEconomy.md`.

---

## Step 1: Inventory

- Map all monetization in the project.
- Find GamePasses, Developer Products, Premium use, ads, and policy-sensitive monetization.
- If access is partial, ask the user to fill inventory gaps.

## Step 2: Attach References

- Read `references/topics/MonetizationAndEconomy.md` for the core monetization rules.
- Read `references/bridges/ImplementationPatterns.md` for technical patterns when needed.
- Read `references/topics/TestingQA.md` for verification and regression coverage.
- Read `references/topics/CommunitySafety2026.md` when ads or policy matter.

## Step 3: Audit Order

1. Receipt safety.
2. Server-owned grant flow.
3. Duplicate protection and idempotency.
4. Pricing and value clarity.
5. Premium integration.
6. Ads and policy fit.
7. Ethical risk review.

## Step 4: Report

- Lead with receipt safety.
- List current revenue streams.
- List risks and fixes.
- Order recommendations by impact-to-effort ratio.
