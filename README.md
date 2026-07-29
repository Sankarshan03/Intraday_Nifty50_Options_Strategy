# Nifty 50 Options — Three-Sleeve Intraday Strategy

A systematic trading strategy for Nifty 50 options that combines three distinct signal sleeves — **Mean Reversion**, **Trend Following**, and **Semi-Directional** — using an ensemble approach with dynamic weighting.

## Strategy Overview

The strategy operates on intraday minute-bar data and generates daily signals based on:

| Sleeve | Trigger | Direction |
|--------|---------|-----------|
| **Mean Reversion** | Z-score < -1 or > +1 on open | Fade the move |
| **Trend Following** | EWM crossover + momentum + breakout strength | Follow the move |
| **Semi-Directional** | Open gaps above prev-high / below prev-low by ≥ 0.2 ATR | Follow gap direction |

### Ensemble Blends

| Blend | Composition | Use Case |
|-------|-------------|----------|
| **Static** | 45% MR / 0% TR / 55% SD | Reporting baseline |
| **Live** | 20% Static + 80% Inverse-Volatility | Primary execution |
| **Returns-Weighted** | Dynamic weights from 60-day trailing returns | Alternative approach |

## 📈 Performance Summary (2025–2026)

| Strategy | Trades | Win Rate | CAGR | Sharpe | Sortino | Max DD | Calmar |
|----------|--------|----------|------|--------|---------|--------|--------|
| Mean Reversion | 104 | 51.9% | 3.01% | 0.80 | 0.86 | 3.46% | 0.87 |
| Trend Following | 45 | 51.1% | 0.07% | 0.05 | 0.04 | 1.80% | 0.04 |
| Semi-Directional | 51 | 72.5% | 10.43% | 3.33 | 4.49 | 1.62% | 6.43 |
| **Ensemble — Static** | 133 | 63.2% | 7.08% | 3.38 | 4.56 | 0.78% | **9.05** |
| **Ensemble — Live** | 138 | 58.0% | 4.12% | 2.53 | 3.65 | 1.14% | 3.62 |
| Ensemble — Returns-Weighted | 116 | 58.6% | 6.71% | 2.40 | 3.11 | 1.56% | 4.29 |

### Walk-Forward Performance (Static Blend)

| Window | Period | Days | CAGR | Max DD | Calmar |
|--------|--------|------|------|--------|--------|
| W1 | 2025-01-01 → 2025-03-21 | 55 | +12.69% | 0.51% | 24.98 |
| W2 | 2025-03-24 → 2025-06-16 | 56 | +20.60% | 0.78% | 26.30 |
| W3 | 2025-06-17 → 2025-09-04 | 56 | +0.38% | 0.76% | 0.51 |
| W4 | 2025-09-05 → 2026-05-15 | 56 | +3.36% | 0.58% | 5.74 |

**Mean Calmar = 14.38 | Std = 11.42**

## Data Requirements

- **Source:** `nifty_options_chain_clean` Parquet file
- **Period:** 2023-01-02 to 2026-05-15 (702 trading days)
- **Format:** Minute bars with OHLCV, expiry, option type, and strike data
- **Size:** ~91 million rows

### Key Data Processing Steps

1. **Synthetic Spot** — Calculated via put-call parity on front-month ATM options
2. **Intraday Features** — ATR (14-bar), AEMA (adaptive EMA)
3. **Daily Aggregation** — OHLC, ATR, trend indicators, Z-scores
4. **Signal Generation** — Three independent signal streams
5. **Ensemble Blending** — Dynamic and static weight combinations

## Quick Start

### Prerequisites

```bash
pip install numpy pandas matplotlib scipy fastparquet
```

### Running the Notebook

1. **Update data path** in the configuration cell:
```python
PARQUET_PATH = r'path/to/your/nifty_options_chain_clean.parquet'
```

2. **Run all cells** sequentially

3. **Outputs** will be saved as:
   - `strategy_returns_log.csv` — Daily P&L and equity curves
   - `strategy_metrics_summary.csv` — Performance metrics
   - `equity_curves.png` — Visualization

### Optional Regime Diagnostic

Requires `statsmodels`:
```bash
pip install statsmodels
```
Computes IV term structure and Hurst exponent to analyze market regimes.

## Output Files

### strategy_returns_log.csv

Contains daily P&L for all sleeves and ensemble blends, including:
- Individual sleeve returns (`mean_pnl`, `trend_pnl`, `semi_pnl`)
- Ensemble blend returns (`pnl_static`, `pnl_iv`, `pnl_rw`, `pnl_live`)
- Dynamic weights (`w_iv_*`, `w_rw_*`)
- Equity curves (`eq_*`)

### strategy_metrics_summary.csv

Performance metrics for all strategies:
- Trades, Win Rate, CAGR, Sharpe, Sortino, Max DD, Calmar

### equity_curves.png

Visualization of equity curves for all strategies over the test period.

## Key Technical Details

### Feature Engineering
- **ATR (14-bar)**: Average True Range for volatility measurement
- **AEMA**: Adaptive EMA with alpha based on normalized ATR
- **Trend Indicators**: Fast/Slow EWM crossover, momentum, breakout strength
- **Mean Reversion**: 20-day rolling Z-score of closing prices

### Option Pricing
- Black-Scholes for implied volatility calculations
- Put-call parity for synthetic spot construction
- ATM strike selection via minimum distance to spot

### Risk Management
- Gap-contamination guard (avoids signals near trading gaps)
- Inverse-volatility weighting for ensemble blending
- No leverage (100% equity baseline)

## Important Notes

1. **No transaction costs** are included in reported returns
2. **Warm-up period**: 2023-2024 data used for feature initialization
3. **Test window**: 2025-01-01 to 2026-05-15
4. **Gap-contaminated rows**: 38 rows removed from MR and Trend signals
5. **Risk-free rate**: 7% INR overnight proxy
