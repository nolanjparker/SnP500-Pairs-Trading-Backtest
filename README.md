# Pairs Trading Statistical Backtest

A Python backtesting project that identifies cointegrated S&P 500 pairs using
historical membership records and OHLCV data and trades deviations from their
historical relationship.

## Overview

This project identifies potentially cointegrated stock pairs during a
formation period, constructs a mean-reverting spread, and simulates trading
over subsequent out-of-sample periods.

The backtest includes transaction costs, position sizing, leverage tracking,
and performance metrics such as Sharpe ratio and maximum drawdown.

## Strategy

1. Select candidate stock pairs.
2. Test pairs for cointegration during the formation window.
3. Estimate the hedge ratio and spread.
4. Standardize the spread using a z-score.
5. Enter positions when the spread exceeds an entry threshold.
6. Exit when the spread mean-reverts or the holding period exceeds a specified maximum.
7. Repeat using rolling/walk-forward formation and trading windows.

## Backtesting Methodology

- Historical OHLCV data: Daily stock price data from Yahoo Finance via `yfinance`, with historical S&P 500 membership data from [S&P 500 Historical Constituents](https://github.com/pierrebrunelle/sp500-historical-constituents) to reduce survivorship bias.
- Formation period: 504 trading days (~2 years).
- Trading period: 1 month, after which pairs and formation-period parameters are re-estimated.
- Pair selection: Candidate pairs are first filtered based on correlation, then tested for cointegration during each formation window.
- Multiple-testing correction: The backtest is evaluated both with unadjusted cointegration p-values and with Benjamini-Hochberg correction to reduce false discoveries caused by testing many candidate pairs.
- Transaction costs: 5 basis points (0.05%) per dollar traded.
- Execution assumptions: Signals are generated using closing prices and trades are executed at the next trading day's open.
- Position sizing: Each trade is scaled to $10,000 of gross exposure, with the two legs sized according to the estimated hedge ratio.

The backtest calculates:

- Cumulative P&L
- Sharpe ratio
- Maximum drawdown
- Peak gross leverage
- Number of trades
- Total net P&L
- Average trade return
- Median trade return
- Win rate
- Average holding period
- Mean-reversion exit rate
- Profit factor

## Results

The strategy was not ultimately successful for either the raw p-value dataset or the Benjamini-Hochberg (BH)-corrected dataset. Though BH-correction did increase stability and ultimately result in positive net cumulative P&L, the strategy resulted in a negative Sharpe ratio for both datasets. Trades that mean-reverted provided high profit-margins, but trades that exceeded the maximum holding period were frequent and had negative profit margins that significantly decreased the overall cumulative P&L. It is also worth noting that the strategy was greatly disrupted for both datasets during the 2020-2022 period (perhaps related to COVID market conditions), though the BH-corrected dataset ultimately recovered and actually ended with higher net P&L afterwards.

| Metric | BH-Corrected Strategy | Raw P-Value Strategy |
|---|---:|---:|
| Cumulative P&L | $20,525.58 | -$420,450.78 |
| Sharpe Ratio | -1.72 | -0.39 |
| Maximum Drawdown | 2.76% | 48.87% |
| Peak Gross Leverage | 1.04× | 16.34× |
| Number of Trades | 160 | 4,527 |
| Closed-Trade Net P&L | $20,524.18 | -$389,855.80 |
| Average Trade Return | 1.28% | -0.86% |
| Median Trade Return | 3.29% | 2.91% |
| Win Rate | 71.88% | 70.89% |
| Average Holding Period | 187.44 days | 174.87 days |
| Mean-Reversion Exit Rate | 67.50% | 68.94% |
| Profit Factor | 1.58 | 0.79 |

### Cointegrated Pairs with Raw P-Values

![Raw P-Value Cointegrated Pairs](assets/raw_cointegrated_pairs.png)

### Cointegrated Pairs with Benjamini-Hochberg Correction

![Benjamini-Hochberg Cointegrated Pairs](assets/bh_cointegrated_pairs.png)

### cumulative P&L Curve

![Cumulative P&L](assets/bh_vs_raw_cumulative_pnl.png)

## Project Structure

```text
SnP500-Pairs-Trading-Backtest/
├── assets/                             # Output graphs
│   ├── bh_cointegrated_pairs.png
│   ├── bh_vs_raw_cumulative_pnl.png
│   └── raw_cointegrated_pairs.png
├── data/                               # Not tracked in Git
│   ├── membership_snapshots/           # Monthly historical S&P 500 membership records
│   ├── bh_corrected_results.parquet    # BH-corrected cointegrated pairs dataset
│   ├── cointegration_results.parquet   # All cointegration results
│   ├── historical_ohlcv_df.parquet     # Historical monthly OHLCV data for S&P tickers
│   ├── membership_df.csv               # Formatted CSV of historical S&P membership data
│   └── sig_results.parquet             # Raw p-values cointegrated pairs dataset
├── .env
├── .gitignore
├── 01_data_collection.ipynb            # Notebook for generating required data files
├── 02_cointegration_testing.ipynb      # Notebook for identifying cointegrated pairs
├── 03_strategy_backtest.ipynb          # Notebook for backtesting and generating statistics/graphs
├── README.md
└── requirements.txt
```

## Installation

Clone the repository and install the required Python dependencies.

```bash
git clone <repository-url>
cd pairs-trading-statistical-backtest
pip install pandas numpy matplotlib statsmodels yfinance jupyter
```

Then launch Jupyter Notebook or open the notebooks in an environment such as VS Code.

```bash
jupyter notebook
```

## Usage

Run the notebooks in numerical order. The project workflow consists of collecting historical market data, identifying candidate pairs, applying correlation filtering and cointegration testing, correcting for multiple hypothesis testing using the Benjamini-Hochberg procedure, estimating strategy parameters, and performing the walk-forward backtest.

The backtest produces performance metrics including cumulative net P&L, Sharpe ratio, maximum drawdown, peak leverage, number of trades, win rate, average and median trade return, average holding period, mean-reversion rate, and profit factor.

Generated figures can be saved to the `assets/` directory for inclusion in the README.

## Limitations

- Historical performance does not guarantee that the strategy would remain profitable in live trading.
- Transaction costs are modeled, but real-world execution may also be affected by slippage, bid-ask spreads, liquidity, and market impact.
- Cointegration and historical correlation relationships can break down over time.
- Results may be sensitive to strategy parameters such as entry thresholds, exit thresholds, formation-window length, and trading-window length.
- The backtest cannot fully reproduce the execution conditions that would occur in a live market.
- Testing many securities and parameter choices can increase the risk of overfitting despite the use of multiple-testing correction.

## Future Improvements

- Add more realistic models for slippage, bid-ask spreads, and market impact.
- Test the robustness of the strategy across different parameter values and market regimes.
- Expand the strategy to additional asset classes or larger security universes.
- Add additional portfolio-level risk controls and position-sizing methods.
- Refactor portions of the notebook-based implementation into reusable Python modules.
- Investigate alternative pair-selection and mean-reversion models.

## Technologies

- **Python** — primary programming language
- **pandas** — data manipulation and time-series processing
- **NumPy** — numerical calculations
- **statsmodels** — statistical tests, cointegration analysis, and multiple-testing correction
- **yfinance** — historical market data
- **Matplotlib** — performance and strategy visualizations
- **Jupyter Notebook** — research, development, and backtesting environment
- **Git / GitHub** — version control and project hosting