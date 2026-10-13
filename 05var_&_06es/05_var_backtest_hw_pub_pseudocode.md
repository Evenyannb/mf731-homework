
ALGORITHM: VaR Backtesting of S&P 500
============================================================


INPUT:
    S&P 500 daily closing prices (2016-05-23 to 2026-05-22)
    Confidence level α = 0.95
    Significance level β = 0.05
    EWMA parameters λ = θ = 0.97
    Rolling window M = 1010

OUTPUT:
    Historical Simulation VaR estimates
    Normal-EWMA VaR estimates
    Daily exceedance indicators
    Total exceedances for each method
    95% confidence interval
    Backtesting conclusions


--------------------------------------------------
TASK 1: DATA COLLECTION AND PREPARATION
--------------------------------------------------

1. Download daily S&P 500 closing prices.

2. Remove missing observations.

3. Let N = number of closing price observations.

4. Display:
       First 5 observations
       Last 5 observations
       Total number of observations N

5. Compute daily logarithmic returns:

       X[t] = ln(S[t] / S[t-1])

6. Compute daily portfolio losses using
   the full loss operator:

       L[t] = 1 - exp(X[t])

7. Let:
       T = number of log returns
       J = T - M

   In this implementation:
       T = 2514
       J = 1504


--------------------------------------------------
TASK 2: HISTORICAL SIMULATION VaR
--------------------------------------------------

8. FOR t = M TO T-1:

       a. Select previous M daily losses:

              Window = L[t-M : t]

       b. Calculate empirical VaR:

              VaR_HS[t] = Quantile(Window, α)

       c. Obtain actual loss:

              Actual_Loss = L[t]

       d. Determine exceedance:

              IF Actual_Loss > VaR_HS[t]:
                  Exceed_HS[t] = 1
              ELSE:
                  Exceed_HS[t] = 0

       e. Store VaR_HS[t] and Exceed_HS[t].

   END FOR

9. Calculate:

       K_HS = SUM(Exceed_HS)

       Rate_HS = K_HS / J

10. Plot:
       Actual daily losses
       Historical VaR estimates
       Exceedance points


--------------------------------------------------
TASK 3: NORMAL-EWMA VaR
--------------------------------------------------

11. Initialize parameters using the
    first M daily log returns:

       μ = SampleMean(X[0:M])

       σ² = SampleVariance(X[0:M])

12. FOR t = M TO T-1:

       a. Calculate current volatility:

              σ = SQRT(σ²)

       b. Compute normal-model VaR:

              z = Φ_inverse(1 - α)

              VaR_EWMA[t] = 1 - exp(μ + σ*z)

       c. Observe actual portfolio loss:

              Actual_Loss = L[t]

       d. Determine exceedance:

              IF Actual_Loss > VaR_EWMA[t]:
                  Exceed_EWMA[t] = 1
              ELSE:
                  Exceed_EWMA[t] = 0

       e. Store VaR_EWMA[t]
          and Exceed_EWMA[t].

       f. Observe current log return:

              x = X[t]

       g. Update conditional variance:

              NewVariance =
                  θ*σ² + (1-θ)*(x-μ)²

       h. Update conditional mean:

              NewMean =
                  λ*μ + (1-λ)*x

       i. Update parameters:

              σ² = NewVariance
              μ = NewMean

   END FOR

13. Calculate:

       K_EWMA = SUM(Exceed_EWMA)

       Rate_EWMA = K_EWMA / J

14. Plot:
       Historical Simulation VaR
       Normal-EWMA VaR


--------------------------------------------------
TASK 4: BINOMIAL BACKTESTING
--------------------------------------------------

15. Set theoretical exceedance probability:

       p = 1 - α = 0.05

16. Assume under the null hypothesis:

       K ~ Binomial(J, p)

17. Calculate expected exceedances:

       Expected_K = J*p

18. Construct the 95% interval:

       Lower = BinomialQuantile(β/2, J, p)

       Upper = BinomialQuantile(1-β/2, J, p)

19. Evaluate Historical Simulation:

       IF Lower <= K_HS <= Upper:
           HS_Result = "Pass"
       ELSE:
           HS_Result = "Fail"

20. Evaluate Normal-EWMA:

       IF Lower <= K_EWMA <= Upper:
           EWMA_Result = "Pass"
       ELSE:
           EWMA_Result = "Fail"

21. Display summary table containing:

       Method
       Total exceedances
       Observed exceedance rate
       Expected exceedance rate
       Whether inside confidence interval


--------------------------------------------------
FINAL RESULTS
--------------------------------------------------

Evaluation periods:
    J = 1504

Expected exceedances:
    75.20

95% interval:
    [59, 92]

Historical Simulation:
    Exceedances = 55
    Exceedance rate = 3.66%
    Result = Fail (Conservative)

Normal-EWMA:
    Exceedances = 94
    Exceedance rate = 6.25%
    Result = Fail (Underestimates Risk)


--------------------------------------------------
END ALGORITHM
--------------------------------------------------
