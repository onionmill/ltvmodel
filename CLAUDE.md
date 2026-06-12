# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Single-file static web app — everything lives in `index.html` (HTML structure, CSS, and JavaScript). No build step, no package manager, no dependencies to install. Deployed via GitHub Pages to `retireonbtc.org` (CNAME).

External dependencies loaded from CDN:
- Chart.js 4.5.1 (`chart.umd.min.js`)
- Google Fonts: Fraunces, Spectral, JetBrains Mono

## Development

Open `index.html` directly in a browser, or serve locally:

```bash
python3 -m http.server 8080
```

No build, lint, or test commands.

## Architecture

The page has two independent sections, each with its own model and charts:

### Section 1 — Borrowing phase (LTV stress model)

Controls: interest rate, annual expense, tranche size, max LTV, horizon year, band multiple.

- `build(p)` — simulates month-by-month liability accrual (tranches drawn when cash runs dry, interest compounded monthly) and records trend/stress collateral values
- `solve(p)` — calls `build`, finds the month where `liabilities / (maxLtv × stressPrice)` is maximised, returns the required BTC stack `N`
- `render()` — reads sliders, calls `solve`, updates result stats and redraws Chart.js charts `ch1` (USD liabilities vs collateral) and `ch2` (LTV over time)

### Section 2 — Accumulation phase (DCA)

Controls: monthly savings, effective price paid multiplier. Shares the Section 1 sliders (rate, expense, tranche, LTV).

- `requiredStackStarting(S, p)` — re-runs the LTV stress simulation from a future start date `S` over a fixed 30-year horizon (using `RATE_DISCOUNT = 2/3` on the interest rate) to find the minimum required stack
- `renderDca()` — walks month-by-month from today, accumulates BTC via DCA, samples `requiredStackStarting` every 3 months (performance), finds the first crossover, updates result stats, and redraws `ch3`

### Power-law model constants

```js
const A = 1.7283e-17, B = 5.76, GEN = Date.UTC(2009, 0, 3); // Bitcoin genesis
function trend(t) { return A * Math.pow((t - GEN) / DAY, B); }
```

Stress price = `band × trend(t)` where `band ≈ 0.42` (Porkopolis 16.5th-percentile floor).

### CSS design tokens

All colors and fonts are CSS custom properties on `:root` (e.g. `--orange: #f7931a`, `--teal: #37c8c3`). The layout uses a two-column CSS Grid above 880px viewport width.

## Deployment

Push to `main` — GitHub Pages deploys automatically from the repo root.
