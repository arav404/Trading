# Brent-Crude Pairs Trading Backtest (CL vs BZ)


This repository contains a pairs trading research notebook that builds and evaluates a simple statistical arbitrage strategy between WTI Crude Oil and Brent Crude Oil using daily data.

The workflow covers: data acquisition, cointegration sanity check, hedge ratio estimation, spread/z-score construction, signal generation (threshold and smooth/tanh), and a basic backtest with performance metrics (Sharpe ratio, confusion matrices/diagnostics).

## Strategy Summary

**Instruments**
- `CL=F` (WTI Crude Oil continuous futures, Yahoo Finance)
- `BZ=F` (Brent Crude Oil continuous futures, Yahoo Finance)

**Core idea**
1. Estimate a hedge ratio via linear regression on the training set.
2. Construct the spread:
   \[
   \text{Spread}_t = BZ_t - (\beta \cdot CL_t + \alpha)
   \]
3. Convert spread to a z-score.
4. Trade mean reversion:
   - If z-score is high: short spread (short BZ, long CL via hedge ratio)
   - If z-score is low: long spread (long BZ, short CL via hedge ratio)

## Implemented Signals

### 1) Threshold (“basic”) signal
- Enter when `|z| > trigger`
- Exit when `z` crosses back toward `0` (exit band)

### 2) Smooth tanh signal
- Uses `-tanh(z / scaling)` to size positions smoothly
- Still respects entry/exit logic to avoid constant flipping

## Backtest Overview

The backtest computes daily PnL from price changes and applies a one-day signal shift to avoid same-day lookahead:

- Position is generated from the signal.
- The strategy PnL is computed using the shifted position and daily price differences.
- A Sharpe ratio is calculated from the resulting daily return series.

## Results (Current State)

- The notebook reports Sharpe ratios on train and test splits and compares the basic vs tanh signal variants.
- Kernel of the approach works end-to-end, but results should be treated as exploratory due to the limitations below.

## Known Limitations and Weaknesses

This project is intentionally a first-pass research implementation. The current version has several important limitations that can materially distort performance:

### 1) Non-causal z-score standardisation
- Z-scores are computed using full-period mean and standard deviation (train and test).
- In live trading, z-score parameters must be computed **causally** (e.g., rolling window), otherwise the strategy uses future information.

### 2) Static hedge ratio
- The hedge ratio is fit once on the full training period.
- The CL–BZ relationship can drift over time; a rolling regression or state-space model is more realistic.

### 3) Costless execution assumption
- No transaction costs, bid/ask spread, slippage, or position limits are modeled.
- Pairs strategies can trade frequently; costs can dominate.

### 4) Continuous futures data caveats
- Yahoo Finance provides continuous futures series which embed roll conventions and may differ from tradable contract PnL.
- Real futures trading requires explicit handling of contract rolls and margin.

### 5) Return/capital definition
- The current “percentage return” calculation uses a denominator that is not a robust proxy for deployed capital or gross notional.
- A cleaner approach is to define a portfolio with explicit notional weights and compute returns from that portfolio value.

### 6) Model selection and validation
- Single train/test split is used.
- A more robust evaluation would be walk-forward or expanding-window validation to reduce regime-specific conclusions.

## What I’m Working On Next

Planned upgrades to make the research closer to a realistic trading backtest:

1. **Rolling/causal z-score**
   - Replace global mean/std with rolling mean/std (e.g., 60–120 day windows).

2. **Rolling hedge ratio**
   - Rolling regression or Kalman filter to adapt to regime shifts.

3. **Realistic return definition**
   - Explicit gross notional allocation and portfolio value tracking.

4. **Transaction costs + slippage**
   - Fixed bps cost per trade and/or spread-based slippage model.

5. **Walk-forward validation**
   - Expanding-window backtest and parameter stability analysis.

6. **Risk management**
   - Volatility targeting, stop logic, max drawdown controls, and exposure constraints.

## Requirements

Python dependencies used in the notebook include:

- `yfinance`
- `numpy`, `pandas`
- `matplotlib`, `seaborn`
- `statsmodels`
- `scikit-learn`

Install (example):

```bash
pip install yfinance numpy pandas matplotlib seaborn statsmodels scikit-learn
