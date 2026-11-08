---
name: factory-review
description: Pre-merge quality and risk gate for any change. Use before merging any PR, especially trading, backtest, risk, or execution changes.
---

# Factory Review — Merge Gate

quality-engineer owns the gate. Trading changes add the three-perspective pass (see cointrader `pr-code-review-trader` pattern).

## General pass (all changes)

1. Correctness, edge cases, error paths, readability.
2. Tests: do they prove the claim? Is the failure case covered?
3. Scope: single purpose, no stowaway refactors.
4. Verdict: APPROVE / APPROVE WITH NITS / REQUEST CHANGES with file:line findings.

## Trading pass (add when strategy, backtest, config, risk, or execution changes)

Apply three lenses and label each finding:

- **SSE:** off-by-one, sign errors, float precision, dead code, config fallback semantics.
- **Architect:** backtest/prod duplication and drift, magic constants, mixed signal/execution concerns, hidden coupling.
- **Quant:** backtest-vs-live gaps (fees, slippage, fill model, data source), risk-guard weakening, capital-efficiency cost, concurrency/double-execution.

## Risk veto

risk-manager verdict required for live-capital changes: APPROVE-FOR-LIVE or VETO with numbered fixes. A VETO blocks merge regardless of other approvals.
