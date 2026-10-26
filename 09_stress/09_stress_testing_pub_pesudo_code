
## Stress Testing: Methodology and Results

### Methodology

We construct a USD 1 million market-cap-weighted portfolio
of Microsoft (MSFT) and Apple (AAPL), using market
capitalizations as of September 1, 2016.

Daily log returns from September 1, 2011, through
September 1, 2016, are used to estimate the mean
vector and covariance matrix via EWMA, with both
decay parameters set to 0.97.

First, we calculate the initial 1-day 95% VaR,
approximate the 10-day VaR using the square-root-of-time
rule, and set regulatory capital equal to three times
the initial 10-day VaR.

We then apply an extreme initial shock:
- AAPL receives a negative five-standard-deviation shock.
- MSFT receives a negative five-conditional-standard-
  deviation shock, conditional on the AAPL shock.

For Days 2 through 10, correlated daily returns are
sampled from multivariate normal distributions whose
means and covariance matrices are updated using EWMA.

We simulate 50,000 paths and calculate the corresponding
10-day linearized portfolio losses using fixed initial
dollar positions.

### Results

(a) Average 10-day portfolio loss: [INSERT RESULT]

(b) Stressed 95% 10-day VaR: [INSERT RESULT]

(c) Frequency exceeding initial 10-day VaR:
[INSERT RESULT]

(d) Frequency exceeding regulatory capital:
[INSERT RESULT]

### Interpretation

The stress test examines how an extreme negative
shock can affect the portfolio's loss distribution
over a 10-day horizon.

The EWMA updates allow the simulated volatility and
correlation structure to respond dynamically to
observed returns.

Comparing the stressed VaR with the initial VaR
helps assess whether the baseline risk estimate
adequately reflects the selected stress scenario.

The exceedance frequencies indicate how often
simulated stressed losses exceed the pre-shock
risk thresholds.
