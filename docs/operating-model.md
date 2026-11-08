# Operating Model

## Principle

One orchestrator, specialist executors, explicit gates. The user talks to delivery-manager; delivery-manager talks to everyone else.

## Roles

- **Delivery Manager** (orchestrator): queue, routing, gates, status. Writes no production code.
- **Product Manager**: what + why + acceptance criteria. No implementation detail.
- **CTO**: feasibility, stack, standards, ADRs. Breaks ties.
- **Solutions Architect**: seams and contracts before code.
- **Senior Engineer**: sliced, tested implementation.
- **Quality Engineer**: merge gate.
- **Technical Writer**: docs hygiene, structure, and lifecycle. Advisory only, never gates.
- **Strategy Designer → Quant Trader → Algo Trader → Risk Manager**: trading chain. Risk Manager vetoes live deployment unconditionally.

## Routing table

| Request | Path |
|---|---|
| Vague idea | delivery-manager → product-manager → cto |
| Feature/change | delivery-manager → product-manager → solutions-architect → senior-engineer → quality-engineer |
| Bug | delivery-manager → senior-engineer (repro) → quality-engineer (regression test) |
| New strategy | delivery-manager → strategy-designer → quant-trader → algo-trader → risk-manager → senior-engineer → quality-engineer |
| Risk/exposure question | delivery-manager → risk-manager (blocking) |
| Docs/hygiene request | delivery-manager → technical-writer (advisory) → quality-engineer (gate) |

## Gates

1. No build without spec + acceptance criteria + feasibility.
2. No merge without quality-engineer verdict.
3. No live trading without risk-manager APPROVE-FOR-LIVE. VETO stops the line.
4. Slices stay small: one PR, one concern, verified before the next.
