# README for Intraday Nifty50 Options Strategy Backtest

## Overview

This notebook implements a comprehensive backtesting framework for three distinct trading strategies on Nifty 50 index options:

1. **Mean Reversion Strategy** - Capitalizes on price deviations from historical averages
2. **Trend Following Strategy** - Captures directional momentum with breakout confirmation
3. **Semi-Directional Breakout Strategy** - Trades breakouts above previous highs or below previous lows

The strategies are combined into an ensemble portfolio using inverse volatility weighting, demonstrating robust performance across different market conditions.

## Key Features

- **Synthetic Spot Construction**: Derives a continuous spot price series from front-month call/put parity
- **Adaptive Technical Indicators**: Uses Adaptive EMA (AEMA) and volatility-normalized signals
- **Daily Signal Generation**: Converts intraday data to daily signals with proper look-ahead bias prevention
- **Ensemble Portfolio**: Combines strategies with inverse volatility weighting for improved risk-adjusted returns
- **Comprehensive Metrics**: Calculates CAGR, Sharpe ratio, Maximum Drawdown, Win Rate, Calmar ratio, and more

## Data Requirements

The notebook expects a Parquet file containing cleaned Nifty 50 options data with the following columns:

| Column | Description |
|--------|-------------|
| `ticker` | Option ticker symbol |
| `bar_minute` | Timestamp of the bar |
| `open/high/low/close` | OHLC prices |
| `volume` | Trading volume |
| `expiry` | Option expiry date |
| `opt_type` | Option type ('C' for Call, 'P' for Put) |
| `strike` | Strike price |
| `feed_era` | Data vendor identifier |
| `is_closing_bar` | Boolean flag for closing bars |

### File Path
```
C:\Users\Admin\Downloads\nifty_options_chain_clean_2023-01-02_to_2026-05-15.parquet
```

## Installation

```bash
pip install numpy pandas matplotlib fastparquet
```

## Notebook Structure

### 1. Data Loading & Filtering
- Loads the options data from Parquet format
- Filters to the most recent 12 months (2025-06-30 to 2026-06-30)
- Displays data summary statistics

### 2. Synthetic Spot Construction
- Identifies front-month expiries for each timestamp
- Uses call-put parity: `Synthetic Spot = Strike + (Call Price - Put Price)`
- Selects the ATM strike closest to the synthetic spot
- Creates OTM option legs for the semi-directional strategy

### 3. Feature Engineering
- Calculates True Range and ATR (Average True Range)
- Implements Adaptive EMA (AEMA) with expanding normalization
- Computes Z-scores, trend strength, and half-life metrics
- **No look-ahead bias**: All features use only historical data

### 4. Strategy Signals (Daily)

#### Mean Reversion Signal
- **Entry**: When Z-score < -1.0 → BUY; when Z-score > 1.0 → SELL
- **Exit**: Next day closing price
- **Rationale**: Prices tend to revert to their historical mean

#### Trend Following Signal
- **Entry**: Requires simultaneous confirmation of:
  - Breakout above/below previous close
  - Positive/negative trend gap (fast > slow EMA)
  - Momentum threshold (|trend_momentum| > 0.15%)
  - Trend strength > 0.35
  - Breakout strength > 0.20
- **Exit**: Next day closing price
- **Rationale**: Captures sustained directional moves

#### Semi-Directional Breakout Signal
- **Entry**: Opening price above previous high → BUY; below previous low → SELL
- **Exit**: Next day closing price
- **Rationale**: Simple breakout strategy for volatility regimes

### 5. Portfolio Construction
- Combines three strategies with inverse volatility weighting (20-day rolling window)
- Ensemble weights: Mean Reversion (45%), Trend Following (15%), Semi-Directional (40%)
- Tracks individual and ensemble equity curves starting from 100

### 6. Performance Metrics

The notebook calculates the following metrics for each strategy and the ensemble:

| Metric | Description |
|--------|-------------|
| Total PnL | Total profit/loss over the period |
| CAGR | Compound Annual Growth Rate |
| Sharpe Ratio | Risk-adjusted return (252-day annualized) |
| Max DD | Maximum drawdown from peak |
| Win Rate | Percentage of winning trades |
| Avg Trade PnL | Average profit/loss per trade |
| Num Trades | Total number of trades executed |
| Final Equity | Final portfolio value starting from 100 |
| Calmar Ratio | CAGR / Max Drawdown |

## Results Summary

Based on the backtest output:

| Strategy | Total PnL | CAGR | Sharpe | Max DD | Win Rate | Calmar |
|----------|-----------|------|--------|--------|----------|--------|
| Mean Reversion | 1.08 | 2.04% | 0.33 | 3.54% | 49.23% | 0.58 |
| Trend Following | 0.39 | 0.74% | 0.15 | 3.83% | 60.00% | 0.19 |
| Semi-Directional | 3.93 | 7.52% | 1.20 | 3.83% | 60.00% | 1.96 |
| **Ensemble** | **2.20** | **4.17%** | **1.40** | **1.72%** | **57.01%** | **2.42** |

### Key Observations

- **Semi-Directional**: Highest returns and Sharpe ratio among individual strategies
- **Ensemble**: Best risk-adjusted performance with lowest maximum drawdown and highest Calmar ratio
- **Diversification**: Ensemble combines the strengths of all strategies while reducing drawdowns
- **Trade Frequency**: 65-107 trades across strategies, indicating sufficient sample size

## Output Files

The notebook generates two CSV files:

1. **strategy_metrics_summary.csv** - Summary table of all performance metrics
2. **strategy_returns_log.csv** - Daily returns and equity values for all strategies

## Visualization

The notebook plots equity curves for all four strategies (3 individual + ensemble), showing:
- X-axis: Date
- Y-axis: Equity (starting at 100)
- Comparison of cumulative performance over time

## Potential Next Steps

1. **Transaction Costs**: Add realistic transaction costs and slippage
2. **Position Sizing**: Implement dynamic position sizing based on volatility
3. **Optimization**: Fine-tune signal thresholds using walk-forward optimization
4. **Risk Management**: Add stop-loss and take-profit rules
5. **Options Greeks**: Incorporate Delta and Gamma for more accurate P&L
6. **Machine Learning**: Use ML to enhance signal generation


Always validate with live paper trading before deploying with real capital.
