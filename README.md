# Cointegration-Based Pairs Trading Model (LUV/JBLU)

A statistical arbitrage strategy that tests US airline stocks for cointegration and trades mean reversion in the spread of the strongest pair, Southwest (LUV) and JetBlue (JBLU).

📄 **[Read the full write-up (PDF)](Cointegration-Based Pairs Trading Model (LUV_JBLU).pdf)**

## Overview

All 15 possible pairs across six major US airlines (DAL, UAL, AAL, LUV, ALK, JBLU) were tested for cointegration using the Engle-Granger test. LUV/JBLU was the strongest candidate by a wide margin (**p = 0.007**).

Interestingly, UAL/ALK looked like the most closely related pair by eye but tested weakly (**p = 0.77**). This illustrates the key distinction behind the project: two stocks can move together visually without their spread having stable, mean-reverting behaviour. **Correlation is not cointegration.**

## Methodology

- **Data:** daily prices for the six tickers from 2019 onwards, pulled with `yfinance`
- **Train/test split:** the first 70% of the series (2019 to mid-2024) was used to build the strategy; the remaining 30% was held out as an untouched out-of-sample test period
- **Hedge ratio:** estimated by OLS regression on the in-sample data (β = 1.76). The resulting spread stayed anchored around a stable mean throughout the in-sample period, including the 2020 COVID-19 demand shock, when it deviated furthest before reverting
- **Signal:** rolling 30-day z-score of the spread
- **Trading rule:** enter when the z-score crosses ±2 (betting on reversion), exit when it returns to 0
- **Lookahead bias:** positions are shifted forward one day so trades only use information available at the time
- **Costs:** transaction costs included in the backtest

<!-- Add a chart here if you have one, e.g.:
![Spread and z-score signals](images/spread_zscore.png)
-->

## Results (out-of-sample, after transaction costs)

| Metric | Result |
|---|---|
| Sharpe ratio | 0.27 |
| Max drawdown | −10.7% |
| Number of trades | 29 (~2.5 years) |

The strategy was profitable on data entirely unseen during design. Transaction costs had minimal impact given the low trading frequency (roughly one trade every four weeks). The Sharpe ratio is modest rather than exceptional, and the drawdown is significant relative to the strategy's peak gains.

<!-- Add an equity curve here if you have one, e.g.:
![Out-of-sample equity curve](images/equity_curve.png)
-->

## Parameter Sensitivity

To check the result wasn't dependent on one specific configuration:

| Lookback | Entry z | Outcome |
|---|---|---|
| 30-day | 2.0 | **Primary:** Sharpe 0.27, drawdown −10.7%, 29 trades |
| 30-day | 1.5 | Stable: Sharpe 0.30, drawdown −11%, 41 trades |
| 5-day | 2.0 | Threshold never crossed, 0 trades |
| 5-day | 1.0 | Lost money out-of-sample (overtrading on noise, higher cost drag) |

The 30-day lookback captures a more stable signal than the 5-day alternative, and results are not fragile to small changes in the entry threshold. This is modest but meaningful evidence against overfitting to one parameter choice.

## Limitations and Further Work

- 29–41 trades over ~2.5 years is a small sample for strong claims about a repeatable edge
- Testing across more pairs and sectors, with rolling walk-forward validation, would be more rigorous
- Short-borrow costs are not modelled explicitly, only execution costs
- The economic rationale (shared fuel cost exposure, overlapping leisure-travel customers between two budget carriers) is plausible but was not verified against fundamental data

## How to Run

<!-- Replace the filenames below with your actual ones -->
```bash
pip install -r requirements.txt
python pairs_trading.py
```

## Tech

Python · pandas · statsmodels · yfinance · matplotlib
