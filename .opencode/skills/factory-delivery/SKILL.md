---
name: factory-delivery
description: Deliver an approved spec through design, sliced implementation, and verification. Use when a spec is approved and ready to build.
---

# Factory Delivery — Spec to Ship

Route: delivery-manager → solutions-architect → senior-engineer → quality-engineer. Small slices, verified each step.

## Steps

1. **Design.** solutions-architect adds components, interfaces, data contracts, error paths to the spec. Record lasting decisions as ADRs.
2. **Slice.** Break into thin vertical slices (one PR each). First slice is the skeleton walking the full path.
3. **Build (per slice).** senior-engineer: failing test → minimal code → refactor → repo pre-commit gates (tests, lint, typecheck/clippy/fmt per repo AGENTS.md).
4. **Verify.** quality-engineer: APPROVE / REQUEST CHANGES. No merge on red.
5. **Track.** delivery-manager reports: done / next / blocked after every slice.

## Rules

- One concern per PR. Stacked branches for dependent slices, targeting each other — only the first targets main.
- Touch only what the slice requires. No drive-by refactors.
- Trading slices: quant-trader checks math, algo-trader checks execution, risk-manager gates live risk. risk-manager VETO stops the line.
