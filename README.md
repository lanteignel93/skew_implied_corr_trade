# Skew Implied-Correlation Trade — SPX Dispersion via the Wing Correlation Premium

> [!NOTE]
> **This is the public showcase repository.** To request access to the private, full-source repository, please email [laurent.lanteigne@gmail.com](mailto:laurent.lanteigne@gmail.com).

A systematic two-leg dispersion book that harvests the *wing implied-correlation
premium*: SPX out-of-the-money puts price equity correlation far above both
at-the-money implied correlation and subsequent realized correlation — a
persistent flow rent from index-level hedging demand. The book sells that
premium on the index side and buys back the crash risk through a basket of
single-name puts whose gamma more than pays for itself.

## Hypothesis

Index put skew embeds a correlation premium that is monotone in wing depth
(deeper OTM ⇒ richer implied correlation) and has been **positive every single
trading day for ten years** — while the ATM correlation premium has been dead
since 2018. The edge therefore lives *only* in the wing. Harvesting it requires
solving the two classic dispersion failure modes: the crash (solved with an
intraday breach stop — momentum through a breached strike is confirmed three
independent times in this research program) and the slow correlated bear
(budgeted as a cost of business; conditioning cannot remove it).

## Results (per \$100-vega unit, gross)

| Metric | Development (2015-07 → 2024-07) | Holdout (2024-08 → 2026-07, sealed) |
|---|---|---|
| Per-cycle Sharpe | **+1.68** (107 cycles) | **+2.46** (24 cycles) |
| Mean / cycle | +\$323 | +\$360 |
| Worst cycle | −\$1,078 | −\$697 |
| CAPM alpha (daily, HAC) | **87% of P&L, t = 4.6** | — |
| Per-cycle correlation to SPX | **+0.03** (legs offset: +0.40 / −0.40) | — |
| Bootstrap P(losing year) | 1.8% | — |

Net of baseline transaction-cost assumptions: ≈ 1.1–1.2 dev Sharpe. The
intraday-stop execution uplift (~+12% of index carry) is not included above.

## Strategy Design

- **Index leg**: short SPX ~3-month 25Δ put, naked, monthly roll, sized to
  fixed vega per cycle
- **Breach management**: intraday touch-stop at the strike, then flat until
  the next scheduled roll — no re-entry, no roll-down (both tested, worse);
  stops replace hedging (hedging pays the premium away)
- **Delta policy**: half the smile delta hedged daily; the retained half is a
  deliberate ERP/shadow-delta tilt
- **Names leg**: long 1-month 35Δ puts on the 50 largest S&P names,
  equal-weight, vega-flat vs the index leg, fully delta-hedged per name —
  pure gamma/vol position and crash convexity; never interrupted by the stop
- **Sizing**: flat vega per cycle — Kelly-optimal here because the premium
  scales with cycle variance (μ ∝ σ²) across every measured regime; nine
  registered conditioning rules all failed to beat flat out of sample

## Research Protocol

- Every grid, breach rule, regime family, and sizing rule **pre-registered
  before results**, with kill criteria
- 2024-08+ holdout sealed through all development; usage itemized (two
  authorized spec looks + one sizing-overlay umbrella), then closed
- Identity-audited backtest library (telescoping, window, and sizing asserts
  on every P&L computation; regression anchors)
- Retractions and negative results kept on the record

## Private Repository Structure

```
skew_implied_corr_trade/
├── cycles.py                 # Audited cycle-P&L library (identity asserts)
├── tests_cycles.py           # Library test suite incl. hand-verified cycle
├── STRATEGY.md               # Final locked specification
├── FINDINGS.md               # §1–26 research trail incl. retractions
├── HYPOTHESIS_*.md           # Pre-registrations (P0, F, S + amendments)
├── PILOT.md                  # Go-live validation plan & kill criteria
├── scripts/                  # 11–24: anchors, grids, breach rules, regimes,
│                             #   sizing OOS tests, decomposition, greeks
├── notebooks/                # PITCH (full proposal), regime/sizing study,
│                             #   position book, trade lifecycle, bakeoffs
└── data/                     # Chains, panels, results parquets
```

## This Repository

| File | Description |
|---|---|
| `strategy_results.ipynb` | Executed results notebook: edge evidence, performance, statistics, market decomposition, regime analysis |
| `data/book_daily.parquet` | Daily P&L per unit (book + legs) and SPX returns |
| `data/cycles.parquet` | Per-cycle P&L with entry VIX/VVIX |
| `data/wing_implied_corr.parquet` | Daily implied correlation by put delta + realized correlation (in-house measurement output) |

The notebook runs top-to-bottom from the bundled parquets — no external data
required. Construction code, data pipeline, and the implied-correlation
measurement methodology live in the private repository.

## License

Private research. Not for redistribution without permission. Results are
research output, gross of costs unless stated, and not investment advice.
