# Spec: Cointrader bear-capitulation gate (spot-only, backtest-only)

Target product repo: `cointrader` (sibling of this factory repo).
This spec lives here per owner direction: all project specs in this repo for now.

## Problem

Cointrader long mean-reversion strategies buy every oversold dip and bleed in
sustained bear markets while grinding below the 200-day average. There is no
spot-only regime that preserves USDC except for extreme capitulation wicks.
We need a defined bear-capitulation entry plus fast-recycle exit with
cash-preservation limits, proven by walk-forward backtest before any live work.

## Non-goals

- No shorting, margin, futures, or leveraged tokens. Spot USDC hold only.
- No live Lambda deploy in V1. Backtest plus paper runner only.
- No new exit class in V1. Reuse `GenericExitEvaluator` with tightened table.
- No per-asset RSI tuning in V1. Single `RSI_OVERSOLD_THRESHOLD=30`.
- No ATR sizing or ranking filter in V1. Fixed `TRADE_VALUE` with 0.5x/1.5x tiers.

## Users

- Strategy researcher running the bear-capitulation backtest and optimisers.
- Strategy operator running the paper screener and reviewing trade logs.
- Reviewers (`quality-engineer`, future `risk-manager` for live promotion).

## Acceptance criteria

- Given a 1095-day backtest on 7-10 USDC pairs with 0.4 percent fee both sides, When the Markov ablation runs, Then both runs emit return, maxDD, win-rate, cash-exposure percent, and a trade log.
- Given the 70/30 walk-forward split, When OOS `totalReturnPct <= 0` or OOS `maxDD >= 30 percent`, Then the env is NOT promoted and the result is flagged OVERFIT.
- Given price below SMA200 with `z` between -2.0 and -4.0 and no Markov capitulation flag, When the screener runs, Then it emits no BUY.
- Given `RSI14 > 30` or `z30 > -2.0`, When the screener runs, Then it emits no BUY.
- Given open positions `>= 3` portfolio-wide, When a new signal fires, Then it is skipped for cash preservation and counted as `cashSkippedEntries`.
- Given `GenericExitEvaluator` thresholds `8/4/2 plus SL 25 percent`, When backtest exits fire, Then fills occur at target or stop price with SL-wins on the same bar and fees deducted both sides.
- Given this repo CI, When spec compliance runs, Then this file contains Problem, Non-goals, Users, Acceptance criteria with Given/When/Then, Risks, and Rollout sections.

## Design

Universe: 7-10 fixed USDC spot pairs on Coinbase, majors plus volatile alts
such as BTC, ETH, SOL plus ZEC, LPT, MATH. No ranking filter.

Entry (all hard, daily 1D):

1. `z30 <= -2.0` using the same z-score helper as the mean-reversion backtest.
2. `RSI14 <= 30`, null blocks entry.
3. Bear gate, either:
   - `close < SMA200` requires `z30 <= -4.0` (2x discount), or
   - Markov regime equals `CAPITULATION` (PendulumMarkov washout state).
4. `MIN_HOURS_BETWEEN_ENTRIES=24`, `MAX_POSITIONS=3` portfolio-wide.
5. Sizing: base `TRADE_VALUE`; 0.5x when below SMA200 without Markov confirm, 1.5x when both confirm.

Exit: `GenericExitEvaluator` proposal `TP_0_2=8`, `TP_3_5=4`, `TP_GT_5=2`,
`STOP_LOSS_PCT=25`, SL-first same-bar.

Backtest plan:

```text
BACKTEST_DAYS=1095, LOOKBACK_DAYS=210, daily-close execution
TRADING_FEE=0.004 both sides
config/backtest-bear-capitulation.env (PRODUCTS, INITIAL_CAPITAL, TRADE_VALUE,
Z_SCORE_THRESHOLD=-2.0, DOWNTREND_Z=-4.0, RSI_OVERSOLD=30, MAX_POSITIONS=3)
Ablation: Markov filter ON vs OFF, same seed and dates
Report: totalReturn, maxDD, win-rate, avgDays, exit-reason split,
per-asset P and L, cash-exposure percent, cashSkippedEntries
```

## Risks / open questions

- Risk: sparse trades under `z <= -2.0` plus `RSI <= 30` may yield under 30 OOS trades and low power. Mitigation: widen universe, not thresholds, in follow-up.
- Risk: SMA200 include-current bias between live and backtest. Mitigation: lock include-current in both and document it.
- Risk: 0.4 percent round-trip fee drag on the smallest 2 percent TP bucket. Mitigation: optimiser must show average return after fees above zero.
- Risk: Markov detector coupling delays delivery. Mitigation: ship `MARKOV_FILTER_ENABLED` boolean stub so ablation runs regardless.
- Risk: volatile-alt slippage vs daily-close assumption. Mitigation: paper uses ticker snapshot; treat backtest P and L as optimistic by fee plus spread.
- Open: confirm final TP table `8/4/2 plus SL25` vs collection default during delivery slice 1.

## Rollout / rollback

- Rollout: keep this spec on branch `feature/bear-capitulation-spec` as DRAFT PR. Product build lands in `cointrader` behind `feature/bear-capitulation-strategy` as stacked slices: config plus detector, strategy plus paper screener, backtest plus env with ablation flag, optimisers, OOS report.
- No live capital, no Lambda promotion, no sheet writes beyond paper tab in V1.
- Rollback: delete the branch; `cointrader` untouched until its own build branch lands.

## Feasibility

- Decision: BUILD. Markdown-only addition here; product build reuses `LongStrategyBase`, `GenericExitEvaluator`, `RsiCalculator`, and backtest cache patterns. No ADR needed. No `risk-manager` live veto required for backtest and paper only; record the veto gate for any future live promotion.
