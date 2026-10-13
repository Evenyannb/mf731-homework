
ALGORITHM: Portfolio Value at Risk (VaR) Estimation
============================================================

INPUT:
    Stock tickers = [MSFT, AAPL, GOOGL]
    Start date = 2024-05-23
    End date = 2026-05-22
    Portfolio value V0 = USD 1,000,000

    Market capitalizations (trillion USD):
        MSFT  = 3.109
        AAPL  = 4.536
        GOOGL = 4.640

    Confidence level α = 0.95
    EWMA decay parameters λ = θ = 0.97
    Monte Carlo sample size N = 100,000

OUTPUT:
    Portfolio market-capitalization weights
    Empirical 95% VaR under three loss operators
    EWMA mean vector and covariance matrix
    Normal EWMA 95% VaR under three loss operators
    Comparison of all VaR estimates


------------------------------------------------------------
PART 1: DOWNLOAD AND PREPROCESS HISTORICAL STOCK DATA
------------------------------------------------------------

1. Define the stock tickers and date range.

2. Download daily closing prices from Yahoo Finance
   for all three stocks.

3. Extract the closing price columns:
       MSFT, AAPL, GOOGL

4. Align observations by trading date.

5. Remove observations with missing prices.

6. Display:
       First 5 observations
       Last 5 observations
       Total number of price observations

7. Plot historical daily closing prices.

8. Calculate daily logarithmic returns:

       FOR t = 2 TO T:
           FOR each stock i:
               X[t,i] = ln(S[t,i] / S[t-1,i])
           END FOR
       END FOR

9. Remove the first missing return observation.

10. Store the return matrix X for subsequent calculations.


------------------------------------------------------------
PART 2: CALCULATE MARKET-CAPITALIZATION PORTFOLIO WEIGHTS
------------------------------------------------------------

1. Store the market capitalizations:

       Cap = [3.109, 4.536, 4.640]

2. Calculate total market capitalization:

       TotalCap = SUM(Cap[i])

3. Calculate the weight of each stock:

       FOR i = 1 TO 3:
           w[i] = Cap[i] / TotalCap
       END FOR

4. Verify:

       SUM(w[i]) = 1

5. Calculate the dollar allocation to each stock:

       FOR i = 1 TO 3:
           Allocation[i] = V0 * w[i]
       END FOR

6. Display the portfolio weights and allocations.


------------------------------------------------------------
PART 3: ESTIMATE EMPIRICAL VaR
------------------------------------------------------------

1. Use historical daily log returns X.

2. FOR each historical observation t:

       Calculate the full portfolio loss:

           L_full[t] =
               V0 * (1 - SUM(w[i] * exp(X[t,i])))

       Calculate the linear approximation:

           L_linear[t] =
               -V0 * SUM(w[i] * X[t,i])

       Calculate the quadratic approximation:

           L_quad[t] =
               -V0 * SUM(
                   w[i] * (X[t,i] + 0.5 * X[t,i]^2)
               )

   END FOR

3. Construct three empirical loss distributions:

       L_full
       L_linear
       L_quad

4. Calculate the empirical 95% VaR:

       VaR_full_emp = Quantile(L_full, 0.95)

       VaR_linear_emp = Quantile(L_linear, 0.95)

       VaR_quad_emp = Quantile(L_quad, 0.95)

5. Display the three empirical VaR estimates.

6. Plot the empirical loss distributions and
   indicate the 95% VaR thresholds.


------------------------------------------------------------
PART 4: ESTIMATE MEAN AND COVARIANCE USING EWMA
------------------------------------------------------------

1. Set the EWMA decay parameters:

       λ = 0.97
       θ = 0.97

2. Initialize the mean vector:

       μ = [0, 0, 0]

3. Initialize the covariance matrix:

       Σ = ZeroMatrix(3,3)

4. FOR each daily return vector X[t]:

       Store the previous mean:

           μ_old = μ

       Calculate the deviation from previous mean:

           d = X[t] - μ_old

       Update covariance matrix:

           Σ = θ * Σ + (1-θ) * (d * dᵀ)

       Update mean vector:

           μ = λ * μ_old + (1-λ) * X[t]

   END FOR

5. Store the final estimates as of 2026-05-22:

       μ_EWMA = μ
       Σ_EWMA = Σ

6. Calculate daily volatility for each stock:

       Vol_daily[i] = SQRT(Σ_EWMA[i,i])

7. Calculate annualized volatility:

       Vol_annual[i] = Vol_daily[i] * SQRT(252)

8. Display:

       EWMA mean vector
       EWMA covariance matrix
       Daily and annualized volatility estimates


------------------------------------------------------------
PART 5: ESTIMATE VaR UNDER NORMALITY
------------------------------------------------------------

ASSUMPTION:

    Daily log returns follow a multivariate
    normal distribution:

       X ~ Normal(μ_EWMA, Σ_EWMA)


A. ANALYTICAL LINEAR VaR

1. Calculate expected portfolio linear loss:

       MeanLoss = -V0 * (wᵀ * μ_EWMA)

2. Calculate standard deviation of linear loss:

       StdLoss = V0 * SQRT(wᵀ * Σ_EWMA * w)

3. Obtain the standard normal 95% quantile:

       z = 1.64485

4. Calculate analytical linear VaR:

       VaR_linear_normal = MeanLoss + z * StdLoss

5. Display the analytical linear VaR.


B. MONTE CARLO SIMULATION

1. Set:

       N = 100,000
       RandomSeed = 42

2. Generate N multivariate normal return vectors:

       FOR k = 1 TO N:

           Generate:

               X_sim[k] ~ Normal(μ_EWMA, Σ_EWMA)

       END FOR

3. FOR each simulated return vector k:

       Calculate full loss:

           MC_full[k] =
               V0 * (1 - SUM(w[i] * exp(X_sim[k,i])))

       Calculate quadratic loss:

           MC_quad[k] =
               -V0 * SUM(
                   w[i] *
                   (X_sim[k,i] + 0.5 * X_sim[k,i]^2)
               )

       Calculate linear loss for validation:

           MC_linear[k] =
               -V0 * SUM(w[i] * X_sim[k,i])

   END FOR

4. Obtain the Monte Carlo VaR estimates:

       VaR_full_normal = Quantile(MC_full, 0.95)

       VaR_quad_normal = Quantile(MC_quad, 0.95)

5. Validate the simulation:

       VaR_linear_MC = Quantile(MC_linear, 0.95)

       Compare VaR_linear_MC with VaR_linear_normal

6. Display:

       Full Monte Carlo VaR
       Analytical linear VaR
       Quadratic Monte Carlo VaR

7. Plot simulated loss distributions.


------------------------------------------------------------
PART 6: COMPARE ALL VaR ESTIMATES
------------------------------------------------------------

1. Construct a comparison table:

       Rows:
           Full
           Linear
           Quadratic

       Columns:
           Empirical VaR
           Normal EWMA VaR
           Absolute Difference
           Percentage Difference

2. FOR each loss operator j:

       Difference[j] =
           VaR_normal[j] - VaR_empirical[j]

       PercentageDifference[j] =
           100 * Difference[j] / VaR_empirical[j]

   END FOR

3. Compare the three loss operators:

       Identify the highest and lowest VaR estimates.

       Calculate the relative spread:

           Spread =
               (MaximumVaR - MinimumVaR)
               / FullVaR * 100

4. Compare empirical and Normal EWMA methods:

       Examine the magnitude of differences.

       Identify whether the differences are material.

5. Discuss possible explanations:

       Historical distribution versus normality

       Heavy tails and extreme historical returns

       EWMA weighting of recent observations

       Effect of estimated expected returns

       Linear versus quadratic approximations

6. Plot a grouped bar chart comparing
   the empirical and Normal EWMA VaR estimates.

7. Summarize the main findings.


------------------------------------------------------------
FINAL RESULTS
------------------------------------------------------------

Empirical 95% VaR:

    Full       = USD 21,045.34
    Linear     = USD 21,418.35
    Quadratic  = USD 21,040.90

Normal EWMA 95% VaR:

    Full       = USD 16,664.95
    Linear     = USD 16,869.11
    Quadratic  = USD 16,662.28

KEY FINDINGS:

    1. Normal EWMA VaR is approximately 21%
       lower than empirical VaR.

    2. The full and quadratic VaR estimates
       are nearly identical.

    3. The linear approximation produces
       the highest VaR under both methods.

    4. Distributional assumptions and parameter
       estimation have a greater impact on VaR
       than the choice of loss operator.

END ALGORITHM
