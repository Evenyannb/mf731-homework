
INPUT:
    S&P 500 daily closing prices
    Portfolio value V = $1,000,000
    N = 2555 daily log returns
    Confidence levels alpha = 0.99 to 0.9999
    EWMA lambda = theta = 0.97
    EWMA initialization window = 100 days
    GEV block sizes n = 73 and n = 35

--------------------------------------------------
STEP 1: DATA PREPARATION
--------------------------------------------------

1. Load historical S&P 500 closing prices.
2. Adjust the sample to N + 1 = 2556 prices.

3. FOR each trading day t:
       X[t] = log(P[t] / P[t-1])
       L[t] = -V * X[t]
   END FOR

4. Obtain N = 2555 daily losses.

--------------------------------------------------
STEP 2: HISTORICAL VaR
--------------------------------------------------

1. Sort daily losses in ascending order.

2. FOR each confidence level alpha:
       Historical_VaR[alpha] =
           Empirical_Quantile(L, alpha)
   END FOR

--------------------------------------------------
STEP 3: EWMA NORMAL VaR
--------------------------------------------------

1. Use returns available through Nov 1, 2021.

2. Initialize using first 100 returns:
       mu = Sample_Mean(X[1:100])
       variance = Sample_Variance(X[1:100])

3. FOR each subsequent return X[t]:

       old_mu = mu

       mu = theta * mu
            + (1 - theta) * X[t]

       variance = lambda * variance
                  + (1 - lambda) * (X[t] - old_mu)^2

   END FOR

4. sigma = sqrt(variance)

5. FOR each alpha:
       Normal_VaR[alpha] =
           -V * mu + V * sigma * Phi_Inverse(alpha)
   END FOR

--------------------------------------------------
STEP 4: GEV BLOCK MAXIMA
--------------------------------------------------

FOR block size n IN {73, 35}:

    1. Divide N losses into m = N / n blocks.

    2. FOR each block i:
           M[i] = Maximum_Loss(Block[i])
       END FOR

    3. Normalize block maxima for numerical stability.

    4. Fit GEV distribution using
       Maximum Likelihood Estimation.

       Estimate:
           xi = shape parameter
           mu = location parameter
           sigma = scale parameter

    5. FOR each alpha:

           IF xi != 0:
               GEV_VaR[alpha] =
                   mu + (sigma / xi)
                   * [(-n * log(alpha))^(-xi) - 1]

           ELSE:
               GEV_VaR[alpha] =
                   mu - sigma * log(-n * log(alpha))

       END FOR

END FOR

--------------------------------------------------
STEP 5: GP THRESHOLD EXCEEDANCES
--------------------------------------------------

1. Set threshold:
       u = Historical_VaR(0.95)

2. Select losses exceeding u:
       Exceedances = {L[t] such that L[t] > u}

3. Compute excess losses:
       Y[i] = Exceedances[i] - u

4. Fit Generalized Pareto Distribution
   to Y using Maximum Likelihood.

   Estimate:
       xi = shape parameter
       beta = scale parameter

5. Set F(u) = 0.95.

6. FOR each alpha:

       IF xi != 0:
           GP_VaR[alpha] =
               u + (beta / xi)
               * [((1-F(u))/(1-alpha))^xi - 1]

       ELSE:
           GP_VaR[alpha] =
               u + beta * log((1-F(u))/(1-alpha))

   END FOR

--------------------------------------------------
STEP 6: RESULTS AND COMPARISON
--------------------------------------------------

1. Plot alpha versus VaR for:
       Historical
       EWMA Normal
       GEV (n=73)
       GEV (n=35)
       GP

2. Evaluate all VaRs at alpha = 0.9999.

3. Identify the method with highest VaR.

4. Calculate:
       Difference between two GEV estimates
       Difference between GP and GEV estimates

5. Discuss:
       Tail risk
       Model sensitivity
       Block size effects
       Parameter estimation uncertainty

OUTPUT:
    VaR comparison plot
    VaR estimates at alpha = 0.9999
    Highest-risk model
    EVT comparison and interpretation
