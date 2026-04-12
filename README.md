# ETF Market Risk Modeling
### VaR Estimation · Backtesting · Stress Testing
**iShares ETF Universe · 2010–2023 · FE535 Project**

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## Overview

End-to-end market-risk workflow for an ETF portfolio — from raw price data through formal regulatory-grade VaR model validation.

```
30-ETF Universe
      │
      ├─ Descriptive Risk Analytics  (μ, σ, Sharpe, β, TE, IR)
      │
      ├─ Portfolio Construction  →  Equal-Weight & Min-Variance
      │         IVV (S&P 500) · IYW (Technology) · IYF (Financials)
      │
      ├─ Distribution Diagnostics  (skewness, kurtosis, Jarque-Bera)
      │
      ├─ VaR & ES Estimation  ─── Historical Simulation
      │   rolling 252-day window   ── Parametric Normal
      │                            ── EWMA / RiskMetrics (λ = 0.94)
      │                            ── Bootstrap Monte Carlo
      │
      ├─ Backtesting ──────────── Kupiec (1995) POF Test
      │                           Christoffersen (1998) Independence
      │                           Conditional Coverage (joint)
      │                           Basel II/III Traffic Light
      │
      └─ Stress Testing ────────  Volatility shocks (1.25×, 1.50×, 2.00×)
                                  Historical worst-window analysis
```

---

## Repository Structure

```
fe535-etf-risk-modeling/
├── etf_market_risk_notebook.ipynb   ← Main analysis notebook
├── etf_market_risk_report.pdf       ← Project report
├── etf_prices.csv                   ← Daily adjusted closes, 30 ETFs
├── results/
│   ├── figures/                     ← Charts (auto-created on run)
│   └── tables/                      ← CSV outputs (auto-created on run)
├── .gitignore
└── README.md
```

---

## Notebook Sections

| # | Section | Content |
|---|---------|---------|
| 1 | Configuration | Global constants — tickers, confidence, rolling window |
| 2 | Data Loading | Load, validate, clean 30-ETF price CSV |
| 3 | Return Construction | Simple returns — justified for linear portfolio aggregation |
| 4 | Risk Analytics | Full-universe metrics — μ, σ, Sharpe, β, TE, IR |
| 5 | Portfolio Setup | Equal-Weight and Min-Variance portfolios on IVV · IYW · IYF |
| 6 | Distribution Diagnostics | Skewness, kurtosis, JB test — motivates non-Gaussian VaR |
| 7 | VaR & ES Model Engine | Four rolling models with ES computed alongside each VaR |
| 8 | Rolling Visualisation | VaR vs realized loss, 21-day rolling breach count |
| 9 | Statistical Backtesting | Kupiec · Christoffersen · Conditional Coverage · Basel TL |
| 10 | Stress Testing | Vol shocks + historical worst-window identification |
| 11 | Model Scorecard | Preferred / Acceptable / Weak ranking |
| 12 | Conclusion | Model recommendations and extensions |

---

## Methods

### VaR Models

| Model | Assumption | Formula |
|-------|-----------|---------|
| **Historical Simulation** | None | −Q₁%(252-day window) |
| **Parametric Normal** | i.i.d. Gaussian | −(μ̂ + z₁% σ̂) |
| **EWMA / RiskMetrics** | Normal, time-varying σ | z₁% × σ̂_EWMA, λ = 0.94 |
| **Bootstrap Monte Carlo** | None (empirical resample) | −Q₁%(10,000 bootstrap draws) |

### Backtests

| Test | H₀ | Statistic |
|------|----|-----------|
| **Kupiec POF** | Observed violation rate = 1% | LR_uc ~ χ²(1) |
| **Christoffersen Independence** | Violations are serially independent | LR_ind ~ χ²(1) |
| **Conditional Coverage** | Correct rate AND independence | LR_cc = LR_uc + LR_ind ~ χ²(2) |
| **Basel Traffic Light** | ≤ 4 exceptions in last 250 days | Green / Yellow / Red |

---

## Setup

```bash
# Clone
git clone https://github.com/Mrinal-g/fe535-etf-risk-modeling.git
cd fe535-etf-risk-modeling

# Install dependencies
pip install numpy pandas scipy matplotlib seaborn

# Run
jupyter notebook etf_market_risk_notebook.ipynb
```

`results/figures/` and `results/tables/` are created automatically on first run.
`seaborn` is optional — the notebook falls back to `matplotlib` if not installed.

---

## Data

| Field | Detail |
|-------|--------|
| File | `etf_prices.csv` |
| Source | iShares / Yahoo Finance |
| Frequency | Daily adjusted closing prices |
| Period | 2010-01-01 to 2023-09-30 |
| Columns | Date + 30 ETF tickers |

---

## Key Results

**ETF selection rationale:** IYW (Technology) and IYF (Financials) have divergent rate sensitivities — Technology benefits from low rates, Financials from high rates. This gives IYW–IYF correlation ~0.70 vs ~0.89 for the other pairs, creating genuine diversification.

**Distribution:** All series reject normality (Jarque-Bera p ≈ 0) with negative skewness and positive excess kurtosis. This predicts that Parametric Normal VaR will over-count violations — confirmed in backtesting.

**Backtest summary:**

| Model | Grade | Reason |
|-------|-------|--------|
| Historical Simulation | **Preferred** | Passes all tests; fully non-parametric |
| Bootstrap Monte Carlo | **Preferred** | Non-parametric; passes all tests |
| EWMA | **Preferred** | Faster vol adaptation; passes all tests |
| Parametric Normal | Acceptable | Violation rate slightly above 1% — fat-tail effect |

---

## License

MIT © Mrinal Gupta
