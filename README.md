# Multi-Factor Regression Model

A small project exploring how much of an asset's return can be explained by well-known systematic risk factors, using the Fama-French 5-factor model plus Momentum (6 factors total).

## Background

Not all of a stock's return is random. Some of it is compensation for taking on specific, well-documented sources of risk — being small-cap, being "cheap" relative to fundamentals, having strong recent momentum, and so on. A factor regression tries to split an asset's return into:

- the part explained by its exposure to each of these known risk factors, and
- the part left over (**alpha**) that the factors can't account for.

### CAPM, the starting point

The original version of this idea was the Capital Asset Pricing Model, which said the *only* risk that matters is exposure to the overall market:

$$E[R_i] - R_f = \beta_i \left(E[R_m] - R_f\right)$$

where $R_i$ is the asset's return, $R_f$ is the risk-free rate, $R_m$ is the market return, and $\beta_i$ measures the asset's sensitivity to the market.

### Fama-French: adding more factors

CAPM alone doesn't explain returns very well — small-cap and "value" stocks persistently outperform what their market beta would predict. Fama and French built this into the model directly, by constructing extra factors as long-short portfolios:

| Factor | Meaning |
|---|---|
| **Mkt-RF** | Market return minus the risk-free rate |
| **SMB** | Small Minus Big — small-cap return minus large-cap return |
| **HML** | High Minus Low — value stocks minus growth stocks (book-to-market) |
| **RMW** | Robust Minus Weak — high profitability minus low profitability |
| **CMA** | Conservative Minus Aggressive — low investment minus high investment |
| **MOM** | Up Minus Down — recent winners minus recent losers (Carhart, 1997) |

The full regression run on each asset's **excess** return (return minus risk-free rate) looks like:

$$R_i - R_f = \alpha + \beta_{Mkt}(R_m - R_f) + \beta_{SMB} \cdot SMB + \beta_{HML} \cdot HML + \beta_{RMW} \cdot RMW + \beta_{CMA} \cdot CMA + \beta_{MOM} \cdot MOM + \epsilon$$

Each $\beta$ is the asset's exposure to that factor. $\alpha$ is whatever's left over after accounting for all six — if it's meaningfully positive, that's return the model can't explain away as compensation for known risks.

## What's in the notebook

- **Data**: monthly returns for `VTI` (total market), `VLUE` (value ETF), `MTUM` (momentum ETF), and `AAPL` (a single stock, included as a less "factor-designed" comparison), pulled via `yfinance`. Factor data comes from `getFamaFrenchFactors`, which sources Ken French's data library.
- **Full-sample regression**: one set of factor loadings + alpha per asset, run via `statsmodels` OLS.
- **Regression diagnostics**: R² (how much of the return the model explains) and t-stats (which factor exposures are statistically meaningful) for each asset.
- **Rolling regression**: the same regression re-run on a rolling 36-month window, to see how factor exposures — and how well the model fits — drift over time. Useful for spotting periods where an asset's risk profile changed, or where factor relationships broke down (e.g. broad sell-offs like the 2020 COVID crash).

## Setup

```bash
pip install -r requirements.txt
```

Then run `factor_model.ipynb` top to bottom — it pulls fresh data each time, so there's no external data file to manage.

## Notes / caveats

- `VLUE` and `MTUM` are themselves factor-tilted ETFs, so their heavy loadings on HML and MOM respectively aren't surprising — that's close to their explicit mandate.
- This is a monthly, full-history regression on a handful of ETFs/stocks — not a robust signal for trading, more a way to build intuition for how factor exposures are estimated and how they move over time.
- Rolling betas are noisy over short windows; 36 months is a common default but is itself a judgment call.

## Credit

Inspired by [this YouTube walkthrough](https://www.youtube.com/watch?v=UA0aJJ6P7T0) on factor regression in Python, extended with regression diagnostics, an additional comparison asset, and rolling R².
