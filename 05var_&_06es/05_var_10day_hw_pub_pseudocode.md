ALGORITHM: 10-Day Value at Risk Estimation Using Monte Carlo Simulation
------------------------------------------------------------

INPUT:
    Confidence level alpha = 0.95
    Initial stock price S0 = 152.51
    Strike price K = 170
    Option maturity T = 0.292
    Risk-free rate r = 0.00119
    Volatility sigma = 0.4907
    Expected stock return mu = 0.1691
    Time step dt = 1/252
    Simulation paths N = 40,000
    Holding period J = 10 days
    Number of short put options = 100

------------------------------------------------------------
STEP 1: INITIALIZE BLACK-SCHOLES PRICING MODEL
------------------------------------------------------------

FUNCTION BS_Put(S, t):

    tau = T - t

    d1 = [ln(S/K) + (r + sigma²/2) * tau]
         / [sigma * sqrt(tau)]

    d2 = d1 - sigma * sqrt(tau)

    PutPrice = K * exp(-r*tau) * N(-d2)
               - S * N(-d1)

    PutDelta = N(d1) - 1

    RETURN PutPrice, PutDelta

END FUNCTION

------------------------------------------------------------
STEP 2: CONSTRUCT INITIAL DELTA-HEDGED PORTFOLIO
------------------------------------------------------------

Calculate initial put price P0 and delta h0

Set initial money market balance:
    Y0 = -h0 * S0

Calculate initial portfolio value:
    V0 = h0*S0 + Y0 - P0

Generate N × J independent standard normal shocks

------------------------------------------------------------
PART 1: ONE-DAY VAR AND SQUARE-ROOT-OF-TIME RULE
------------------------------------------------------------

FOR each simulation i = 1 to N:

    1. Generate one-day stock price:

       S1 = S0 * exp(
            (mu - sigma²/2)*dt
            + sigma*sqrt(dt)*Z[i,1]
       )

    2. Calculate put price P1 at time dt

    3. Keep initial stock hedge h0 unchanged

    4. Grow money market balance:

       Y1 = Y0 * exp(r*dt)

    5. Calculate end-of-day portfolio value:

       V1 = h0*S1 + Y1 - P1

    6. Calculate loss on 100 short puts:

       Loss_1d[i] = 100 * (V0 - V1)

END FOR

Calculate one-day 95% VaR:

    VaR_1d = 95th percentile of Loss_1d

Apply square-root-of-time rule:

    VaR_10d_sqrt = sqrt(10) * VaR_1d

------------------------------------------------------------
PART 2: FULL 10-DAY MONTE CARLO SIMULATION
------------------------------------------------------------

FOR each simulation i = 1 to N:

    Initialize:
        S = S0
        h = h0
        Cash = Y0

    FOR each trading day j = 1 to J:

        1. Simulate next stock price:

           S_new = S * exp(
                   (mu - sigma²/2)*dt
                   + sigma*sqrt(dt)*Z[i,j]
           )

        2. Grow money market balance:

           Cash = Cash * exp(r*dt)

        3. Calculate new put price and delta:

           P_new, h_new = BS_Put(S_new, j*dt)

        4. IF j < J:

               Rebalance stock position:

               Cash = Cash - (h_new - h)*S_new

               Update hedge:
                   h = h_new

           END IF

        5. Update stock price:
               S = S_new

    END FOR

    Calculate final portfolio value:

        V10 = h*S + Cash - P_new

    Calculate 10-day portfolio loss:

        Loss_10d[i] = 100 * (V0 - V10)

END FOR

Calculate full 10-day VaR:

    VaR_10d_MC = 95th percentile of Loss_10d

------------------------------------------------------------
PART 3: COMPARE VAR ESTIMATES
------------------------------------------------------------

Calculate absolute difference:

    Difference = VaR_10d_sqrt - VaR_10d_MC

Calculate percentage difference:

    Percentage_Difference =
        Difference / VaR_10d_MC * 100

Compare the two estimates.

Explain possible differences due to:

    - Nonlinear put option payoff
    - Gamma exposure
    - Daily delta hedge rebalancing
    - Changes in portfolio risk over time
    - Limitations of square-root-of-time scaling

------------------------------------------------------------
STEP 3: VISUALIZE RESULTS
------------------------------------------------------------

Plot histogram of simulated 10-day portfolio losses

Add vertical reference lines for:
    - Full simulation 10-day VaR
    - Square-root-of-time 10-day VaR

Label axes and display legend

------------------------------------------------------------
OUTPUT:
------------------------------------------------------------

    One-day 95% VaR
    Square-root-of-time 10-day 95% VaR
    Full Monte Carlo 10-day 95% VaR
    Absolute and percentage differences
    Histogram of 10-day loss distribution

END ALGORITHM
