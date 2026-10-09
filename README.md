# Strategy Reality Check

A diagnostic tool that takes a trading strategy's return series and reports the reasons the result is probably not real.

There is no "passed" verdict. Every check is a reason to doubt, because no statistical test can demonstrate that a strategy works — only that it has failed to be ruled out.

## What it checks

| Check | What it catches |
|---|---|
| t-statistic and p-value | a Sharpe ratio indistinguishable from zero |
| Sample length | results measured over too little data to mean anything |
| Deflated Sharpe ratio | the Sharpe inflation caused by testing many strategies and keeping the best |
| Skewness | small frequent gains paid for by rare large losses |
| Kurtosis | fat tails that the Sharpe ratio understates |
| Cost assumption | gross results presented as if they were net |

## Why a Sharpe ratio alone is not a result

A Sharpe ratio has no meaning without the length of the period it was measured over:

```
t = Sharpe × √(years)
```

Below 1.96, the result is not distinguishable from zero at 95% confidence. The tool also reports how many years of data would be required to reach significance at the observed Sharpe, which is frequently longer than a career.

## The deflated Sharpe ratio

Testing many strategies and reporting the best one inflates the result even when none of them has an edge. The expected maximum Sharpe across *N* random strategies grows with *N*, so the winner's Sharpe must be discounted by how hard you searched.

The tool computes that expected maximum (Bailey & López de Prado) and subtracts it. What remains is what survives the search.

## Worked examples

Three synthetic cases with known properties:

```
=== random noise, 2 years ===
  500 days (1.98y) · return 6.8% · vol 15.2% · Sharpe 0.18
   • SHORT SAMPLE — 1.98y. This Sharpe will never reach significance in a practical sample.
   • NOT SIGNIFICANT — t=0.26, p=0.798. Consistent with zero edge.
   • NO COSTS MODELLED — results are gross.

=== best of 50 tested strategies ===
  500 days (1.98y) · return 11.4% · vol 16.2% · Sharpe 0.46
   • NOT SIGNIFICANT — t=0.65, p=0.518. Consistent with zero edge.
   • SELECTION BIAS — 50 strategies tried. Sharpe 0.46 deflates to -1.16 (haircut 1.61).
   • PARAMETER TUNING — 2 parameters fitted. Advisory only.
   • NO COSTS MODELLED — results are gross.

=== 12-year track record ===
  3000 days (11.9y) · return 9.3% · vol 12.7% · Sharpe 0.42
   • NOT SIGNIFICANT — t=1.43, p=0.151. Consistent with zero edge.
```

The second case is the central illustration: a Sharpe of 0.46, selected as the best of 50 attempts, deflates to **−1.16**. The haircut of 1.61 exceeds the measured Sharpe entirely — the winner of a 50-strategy search would be expected to post 1.61 from luck alone, so 0.46 is worse than the search would produce by chance.

The third is the uncomfortable one. Twelve years and a Sharpe of 0.42 still fails the significance test. Markets require far more data than most track records contain.

## Usage

```python
verdict(returns,
        name="my strategy",
        n_strategies_tried=20,   # how many variants you tested before choosing this one
        n_params_tuned=3,        # advisory only
        cost_bps=5)
```

`n_strategies_tried` is the input users are most likely to understate. It should include every variant tested, every parameter value swept, and every idea abandoned along the way.

## Tests

`test_reality_check()` verifies:

- zero-mean noise is not flagged significant
- a strong, long track record is flagged significant
- the t-statistic scales as √n when the sample is extended
- the deflation haircut grows with the number of strategies tried
- no haircut is applied when only one strategy was tested

**What these tests do not establish.** They verify internal consistency and expected behaviour. They do not verify the deflated Sharpe formula against a published reference value. The expected-maximum expression is implemented from the Bailey & López de Prado method but has not been checked against a worked example from the source. Treat the deflation as directionally informative rather than exact.

An earlier version of the test suite failed because the first test compared a zero-return series against a 4% risk-free rate and then asserted the result was insignificant — it was significantly negative, correctly. The tool was right; the test was wrong.

## Limitations

- `n_params_tuned` is reported but not incorporated into the deflation, so the true haircut is larger than shown.
- Assumes returns are independent; serial correlation would inflate the t-statistic further.
- Deflation formula unverified against a published value (see Tests).
- No handling of regime changes, structural breaks, or survivorship bias in the underlying data.

## Running it

Open `reality_check.ipynb` in Google Colab and run the cells in order. Requires `numpy`, `pandas`, `scipy`.

## Reference

Bailey, D. & López de Prado, M. (2014). *The Deflated Sharpe Ratio: Correcting for Selection Bias, Backtest Overfitting and Non-Normality.* Journal of Portfolio Management.
