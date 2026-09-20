# SEC Filing NLP Alpha Engine

A research prototype that asks: **Can language extracted from SEC 10-K/10-Q filings using NLP help explain short-term stock returns?** This is a learning/research project, not a live trading system, and its own results are a documented null finding (see `main.tex` for the full write-up, including limitations).

## What's in this repo

| File | Description |
|---|---|
| `SEC_filing_nlp_alpha_engine.ipynb` | The full Colab notebook: downloads SEC filings, scores them with FinBERT sentiment + a risk-language dictionary + a novelty measure, pulls forward stock returns, runs a cross-sectional regression, builds a composite signal, and backtests a long-short portfolio. |
| `research_dataset.csv` | Filing-level dataset: text features (`sentiment_score`, `sentiment_change`, `risk_language`, `risk_change`, `novelty`, `log_word_count`) joined to 5-day forward returns (`ret_5d`). 230 filings, 19 tickers. |
| `nlp_signals.csv` | `research_dataset.csv` plus standardized (z-scored) features and the composite `alpha_signal`. |
| `portfolio_returns.csv` | Daily long-short/long-only portfolio returns from the backtest (only 2 rows — see Limitations below). |
| `regression_results.txt` | Full OLS regression output (statsmodels, HC3 robust standard errors). |
| `main.tex` | LaTeX writeup of the methodology, results, and — importantly — the limitations of this version of the analysis. |
| `figures/` | Charts referenced in `main.tex`. |

## Method summary

1. Pull 10-K/10-Q filings for 19 large-cap U.S. tickers (2020–2026) from SEC EDGAR.
2. Score each filing's sentiment with **FinBERT** (`ProsusAI/finbert`), sentence-sampled and averaged.
3. Compute a dictionary-based **risk-language** intensity score and a **novelty** score (vocabulary change vs. the firm's prior filing).
4. Pull actual 5-day forward stock returns for each filing date via `yfinance`.
5. Test the relationship with an OLS cross-sectional regression, and separately with a hand-weighted composite signal used to form a long/short portfolio (long the top-quartile signal, short the bottom quartile, within each cross-section).

## Results (short version)

- **Regression: no relationship found.** $R^2 = 0.012$; no feature (sentiment, sentiment change, risk language, novelty, or log word count) is statistically distinguishable from zero.
- **Portfolio backtest: not statistically usable.** The cross-sectional grouping rule requires ≥4 filings on the exact same calendar date to form a long/short trade. Only 2 dates in the whole 2020–2026 sample meet that bar, so the "backtest" has only 2 return observations — nowhere near enough to draw a conclusion in either direction.

## Known limitations (refer to `main.tex`, Section 5, for full detail)

- **Cross-sections are grouped by exact filing date**, which is too granular for only 19 tickers — this is what causes the tiny portfolio sample above, and also causes 66% of filings to be dropped from the sentiment-quintile chart.
- **Look-ahead bias**: the z-scores used to build the trading signal are computed over the whole 2020–2026 sample at once, not a point-in-time rolling window.
- **Survivorship bias**: the ticker list contains only large, currently-successful companies.
- **Small sample**: 230 filings total.
- **No trading costs** (spread, commissions, short-borrow) are modeled.

## Suggested next steps

1. Re-group cross-sections by week/month instead of exact date, and make z-scoring point-in-time (rolling/expanding window) to remove look-ahead bias.
2. Expand the ticker universe substantially and hold out a later time period as an out-of-sample test.
3. Add a proper significance test (e.g. bootstrap) for the long-short spread, and annualize performance metrics correctly.

## Disclaimer

This is a research prototype for learning purposes. It is not investment advice and is not a trading system.
