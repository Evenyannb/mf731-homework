
ALGORITHM: EWMA-Based Monte Carlo Stress Testing

INPUT:
    Historical daily log returns of MSFT and AAPL
    Historical period: 09/01/2011 to 09/01/2016

    Portfolio value V = 1,000,000
    Market cap MSFT = 448.77 billion
    Market cap AAPL = 575.11 billion

    Confidence level alpha = 0.95
    Holding period K = 10 days
    Monte Carlo paths M = 50,000
    EWMA mean decay lambda = 0.97
    EWMA covariance decay theta = 0.97

--------------------------------------------------
STEP 1: PREPARE DATA AND PORTFOLIO
--------------------------------------------------

1. Load historical daily adjusted stock prices.

2. Calculate daily log returns:

       X[t] = log(S[t] / S[t-1])

3. Calculate market-cap portfolio weights:

       w[i] = MarketCap[i] / TotalMarketCap

4. Calculate initial dollar positions:

       position[i] = V * w[i]

5. Keep the initial dollar positions fixed
   throughout the stress test.

--------------------------------------------------
STEP 2: ESTIMATE EWMA MEAN AND COVARIANCE
--------------------------------------------------

1. Initialize:

       mu = [0, 0]
       Sigma = zero 2x2 matrix

2. FOR each historical daily return vector X[t]:

       residual = X[t] - mu

       mu_new = lambda * mu
                + (1-lambda) * X[t]

       Sigma_new = theta * Sigma
                   + (1-theta)
                     * residual * residual^T

       mu = mu_new
       Sigma = Sigma_new

   END FOR

3. Calculate individual stock volatilities:

       sigma[i] = sqrt(Sigma[i,i])

4. Calculate stock return correlation:

       rho = Sigma[1,2] /
             (sigma[1] * sigma[2])

--------------------------------------------------
STEP 3: CALCULATE INITIAL VaR AND CAPITAL
--------------------------------------------------

1. Calculate portfolio expected loss:

       mean_L = -position^T * mu

2. Calculate portfolio loss volatility:

       std_L = sqrt(position^T
                    * Sigma * position)

3. Calculate initial one-day 95% VaR:

       VaR_1d = mean_L + 1.645 * std_L

4. Apply square-root-of-time approximation:

       VaR_10d = sqrt(K) * VaR_1d

5. Calculate regulatory capital:

       Capital = 3 * VaR_10d

--------------------------------------------------
STEP 4: APPLY INITIAL STRESS SHOCK
--------------------------------------------------

1. Apply a negative 5-sigma shock to AAPL:

       x_AAPL = mu_AAPL - 5 * sigma_AAPL

2. Compute MSFT conditional mean given AAPL:

       mu_MSFT_cond =
           mu_MSFT
           + rho * sigma_MSFT / sigma_AAPL
             * (x_AAPL - mu_AAPL)

3. Compute MSFT conditional standard deviation:

       sigma_MSFT_cond =
           sigma_MSFT * sqrt(1-rho^2)

4. Apply a negative 5-conditional-sigma
   shock to MSFT:

       x_MSFT =
           mu_MSFT_cond - 5 * sigma_MSFT_cond

5. Define the Day 1 shock vector:

       shock = [x_MSFT, x_AAPL]

   The same deterministic shock is used
   for every Monte Carlo path.

--------------------------------------------------
STEP 5: MONTE CARLO SIMULATION
--------------------------------------------------

Initialize an empty array Losses.

FOR simulation m = 1 to M:

    1. Initialize path-specific parameters:

           mu_path = mu
           Sigma_path = Sigma

           cumulative_return = [0, 0]

    2. DAY 1:

           X[1] = shock

           cumulative_return += X[1]

           Update mu_path and Sigma_path
           using the EWMA equations.

    3. FOR day k = 2 to K:

           Generate two independent
           standard normal random numbers.

           Construct correlated MSFT and
           AAPL returns using the current
           mean vector and covariance matrix:

               X[k] ~ N(mu_path, Sigma_path)

           cumulative_return += X[k]

           IF k < K:

               Update mu_path using EWMA.

               Update Sigma_path using EWMA.

           END IF

       END FOR

    4. Compute 10-day linearized loss:

           L[m] = -position^T
                  * cumulative_return

    5. Store L[m] in Losses.

END FOR

Note:
    In the Python implementation, all M paths
    are processed simultaneously using
    NumPy vectorization.

--------------------------------------------------
STEP 6: CALCULATE FINAL RESULTS
--------------------------------------------------

(a) Average 10-day portfolio loss:

       AverageLoss = mean(Losses)

(b) Stressed 95% 10-day VaR:

       StressedVaR = percentile(Losses, 95)

(c) Initial VaR exceedance frequency:

       ExceedVaR =
           100 * count(Losses > VaR_10d) / M

(d) Regulatory capital exceedance frequency:

       ExceedCapital =
           100 * count(Losses > Capital) / M

--------------------------------------------------
STEP 7: REPORT RESULTS
--------------------------------------------------

OUTPUT:

    Initial 1-day VaR
    Initial 10-day VaR
    Regulatory capital
    Day 1 stress shock

    (a) Average 10-day portfolio loss
    (b) Stressed 95% 10-day VaR
    (c) Initial VaR exceedance percentage
    (d) Capital exceedance percentage

END ALGORITHM
