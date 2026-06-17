# ARM vs Fixed — Rate-Risk Dashboard

An interactive dashboard that answers one question: **what is the risk profile of a
7/6 ARM relative to a 30-year fixed-rate baseline, and what do a refinance and the
worst-case no-refi paths do to the cost difference?**

It's a single self-contained file — `index.html` — with no install, no server, no
build step, and no data leaving the browser. Open it locally or visit the live
version (GitHub Pages).

> The figures shipped as defaults are a **worked example** (a ~$1.37M jumbo
> purchase), not anyone's actual loan. Every input is editable — replace them with
> your own numbers, or with quotes from your lender's rate sheet and Loan Estimate.

## What it models

The 7/6 ARM and a 30-year fixed run on identical 30-year amortization. **The fixed
is always the reference; every output is a difference against it.**

- **Down payment** — 25% vs 30%, driving the loan amount, the LTV pricing tier, and
  the opportunity cost of the cash freed by putting less down.
- **Closing-cost precompute** — rate→price grids; points cost = `(100 − price)/100 ×
  loan`, interpolated in rate and overridable.
- **ARM reset** — fixed through the year-7 reset, then every 6 months: fully-indexed
  rate = `SOFR + margin`, rounded to ⅛%, clamped by the initial / periodic /
  lifetime caps and the floor.
- **Refi event** — at a chosen year N, with a manual or scenario-pulled rate, a
  fresh-30 or remaining term, and rolled / paid-cash / no-cost handling.
- **Rate-path scenarios** — Benign / Base / Adverse / Severe presets driving two
  *separate* forward curves (SOFR for the reset, the 30-yr fixed for refi
  attractiveness), plus a fully editable custom grid with a velocity warning.
- **Refi-as-hedge guard** — suppresses the refi in any scenario whose fixed rate at
  year N is above a threshold, because refinanceability correlates with the benign
  world; the tool will not credit a rescue that wouldn't exist.

## Primary outputs

- **Cumulative cost-delta curve** (ARM strategy − fixed), nominal or PV, all four
  scenarios overlaid, refi marked, each **crossover year** dotted.
- **Payment timeline** — fixed (flat), the ARM through its reset steps per scenario,
  and post-refi.
- **Year-8 reset ladder** — the reset payment shock vs the fixed, per scenario.
- **Scenario comparison table** — {ARM no-refi, ARM refi@N, fixed} × {four regimes},
  cells = total cost-to-horizon difference + crossover.

## Correctness

The financial spine is validated against deterministic anchors at load
(`selfTest()`); the sidebar shows a **✓ Model validated** badge and the console logs
the result. Verified anchors (30% down on the example, $957,250 loan):

| Anchor | Value |
|---|---|
| ARM P&I @ 5.875% | $5,662.50/mo |
| Fixed P&I @ 6.250% | $5,893.95/mo |
| ARM balance at month 84 | ≈ $856,100 |
| Severe reset P&I (re-amortized) | ≈ $6,932/mo |
| ARM points @ 5.875% / 30% down | ≈ +$708 |
| Fixed points @ 6.25% / 30% down | ≈ −$747 |
| Break-even reset rate | ≈ 6.3% (SOFR < ~3.55%) |

The ~$19,440 seven-year payment cushion is overtaken by the no-refi ARM around year
8.5 (Severe) and year 10.1 (Adverse), and never in Base/Benign.

## Tech

Vanilla JavaScript, one reactive `state` object, hand-built inline-SVG charts, CSS
design tokens. No dependencies, no network calls, no storage. The whole thing is the
one HTML file.

## Disclaimer

Planning aid, not financial advice. The forward-curve grids are stated estimates,
not predictions. Margin, caps, index, floor and reset cadence vary by loan — confirm
the actuals from your promissory Note before treating any reset output as
decision-grade.
