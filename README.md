# Product Engineering Factory

Framework repo for running software work with an agent team. Product code lives in product repos (e.g. `cointrader`, `mean-reversion-analyser`); this repo defines **who does what, in what order, and what gates apply**.

## The team

| Agent | Role | When to use |
|---|---|---|
| `delivery-manager` | Orchestrator — owns queue, delegates, enforces gates | Any multi-step task; your default entry point |
| `cto` | Technical authority, stack, ADRs | Feasibility, stack decisions, disputes |
| `product-manager` | Specs with acceptance criteria | Unclear or disputed requirements |
| `solutions-architect` | Module boundaries, interfaces, contracts | Multi-module design, refactors |
| `senior-engineer` | Sliced TDD implementation | All code changes |
| `quality-engineer` | Merge gate: correctness, tests, readiness | Before any merge/deploy |
| `strategy-designer` | Hypothesis → backtestable strategy spec | New trading ideas |
| `quant-trader` | Math, stats, backtest integrity | Signals, indicators, backtests |
| `algo-trader` | Execution, orders, Lambda ops | Anything placing orders |
| `risk-manager` | Blocking authority on live capital | Limits, stops, exposure — can VETO |
| `devops-engineer` | Change-time ops: serverless deploys, pipelines | Infra work, deploys, pipeline repair |
| `sre-engineer` | Steady-state reliability: logs, metrics, alarms | Diagnosis, alert tuning, runbooks |
| `technical-writer` | Docs hygiene and lifecycle, advisory only | READMEs, specs, ADRs, API docs |

## Org chart

```text
                    ┌──────────────────┐
                    │ delivery-manager │  ◄── you talk to this one
                    │  (orchestrator)  │      queue · routing · gates
                    └────────┬─────────┘
                             │ delegates to all below
        ┌────────────────────┼────────────────────┐
        ▼                    ▼                    ▼
  CORE PIPELINE        OPERATIONS            TRADING CHAIN
  (every feature)      (keep it running)     (strategy work)

  product-manager      devops-engineer       strategy-designer
  (DEFINE: specs       (change-time:            │
   + acceptance        deploys, pipelines,      ▼
   criteria)           SAM/stacks)          quant-trader
        │                                       │
        ▼                                       ▼
  solutions-architect  sre-engineer          algo-trader
  (DESIGN: seams,      (steady-state:            │
   interfaces,          logs, metrics,          ▼
   contracts)          alarms, runbooks)    risk-manager
        │                                   (VETO on live
        ▼                                    capital — blocking)
  senior-engineer
  (BUILD: sliced TDD)
        │
        ▼
  quality-engineer
  (GATE: APPROVE /
   REQUEST CHANGES)

  Spanning authorities: cto (feasibility, stack, ADRs, disputes)
  Alongside everything: technical-writer (docs hygiene — advisory, never gates)
```

To answer your question directly: yes — **Delivery Manager** is the right orchestration role. It owns *flow* (what happens next, who does it, did the gate pass). Product Manager owns *what* (spec), CTO owns *why/what stack* (direction). Use Delivery Manager as your single interface.

## Lifecycle

```text
idea → factory-intake (spec + feasibility) → factory-delivery (design → slice → build → verify) → factory-review (merge gate) → ship
```

Trading path inserts `strategy-designer → quant-trader → algo-trader → risk-manager` before build, and `risk-manager` sign-off before live.

## Usage

1. Open this repo in OpenCode.
2. Ask the delivery-manager to take the task: `Use the delivery-manager subagent to …`.
3. It loads the matching `factory-*` skill, delegates, and reports done/next/blocked.

## Layout

```text
.opencode/agents/        13 role definitions
.opencode/skills/        factory-intake, factory-delivery, factory-review, multi-perspective-review
docs/templates/          spec + plan templates
docs/operating-model.md  roles, routing, gates
```

## Conventions

Modelled on `cointrader` and `mean-reversion-analyser`: `AGENTS.md` as the operating contract, DRAFT PRs, stacked branches, verify-before-merge. See `AGENTS.md` and `docs/operating-model.md`.
