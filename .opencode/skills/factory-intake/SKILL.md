---
name: factory-intake
description: Turn a vague request into a buildable spec with acceptance criteria. Use when starting any new work item, idea, or change request.
---

# Factory Intake — Idea to Spec

Route: delivery-manager → product-manager → cto. No code in this phase.

## Steps

1. **Clarify.** State assumptions explicitly. Ask the minimum questions to resolve ambiguity (problem, user, success metric, constraints).
2. **Spec.** product-manager writes `docs/specs/<slug>.md` from `docs/templates/spec-template.md`:
   problem, non-goals, users, acceptance criteria (Given/When/Then), risks, rollout note.
3. **Feasibility.** cto signs off: build/buy/defer + stack constraints + ADR needed? (yes/no).
4. **Gate.** No spec + acceptance criteria + feasibility = no build. Return to intake.

## Trading intake extras

Strategy ideas go via strategy-designer first (hypothesis → backtest plan), then the normal spec path. Live-capital work always adds risk-manager to the gate list.
