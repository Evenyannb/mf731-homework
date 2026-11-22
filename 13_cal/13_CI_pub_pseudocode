
INPUT:
    Apple daily prices (2016-11-01 to 2018-10-31)
    Portfolio value V0 = 1,000,000
    VaR confidence level alpha = 0.97
    CI significance level beta = 0.02
    Number of simulations M = 125,000

STEP 1: DATA PREPARATION
    Download adjusted daily closing prices for AAPL
    Calculate daily log returns:
        X[t] = ln(P[t] / P[t-1])

    n = number of log returns
    mu_hat = sample mean of X
    sigma_hat = sample standard deviation of X
    S2 = sigma_hat^2

    z = inverse_normal_CDF(1 - alpha)


PART (a): KNOWN MEAN, UNKNOWN VOLATILITY

STEP 2: HISTORICAL VaR
    FOR each log return X[t]:
        Loss[t] = V0 * (1 - exp(X[t]))

    Historical_VaR = quantile(Loss, alpha)


STEP 3: THEORETICAL VaR
    Theoretical_VaR =
        V0 * (1 - exp(mu_hat + sigma_hat * z))


STEP 4: CONFIDENCE INTERVAL FOR VOLATILITY
    df = n - 1

    q_lower = chi_square_inverse_CDF(beta/2, df)
    q_upper = chi_square_inverse_CDF(1-beta/2, df)

    sigma_lower = sqrt(df * S2 / q_upper)
    sigma_upper = sqrt(df * S2 / q_lower)


STEP 5: CONFIDENCE INTERVAL FOR VaR
    VaR_lower_a =
        V0 * (1 - exp(mu_hat + sigma_lower * z))

    VaR_upper_a =
        V0 * (1 - exp(mu_hat + sigma_upper * z))

    OUTPUT:
        Historical_VaR
        Theoretical_VaR
        98% CI = [VaR_lower_a, VaR_upper_a]


PART (b): UNKNOWN MEAN AND VOLATILITY

STEP 6: MONTE CARLO SIMULATION
    FOR m = 1 TO M:

        Sample Y from Chi-square(df)

        sigma_m = sqrt(df * S2 / Y)

        Sample mu_m from Normal(
            mean = mu_hat,
            standard_deviation = sigma_m / sqrt(n)
        )

        VaR[m] =
            V0 * (1 - exp(mu_m + sigma_m * z))

    END FOR


STEP 7: MONTE CARLO RESULTS
    Mean_VaR = average(VaR[1:M])

    VaR_lower_b = quantile(VaR, beta/2)
    VaR_upper_b = quantile(VaR, 1-beta/2)

    OUTPUT:
        Mean_VaR
        98% CI = [VaR_lower_b, VaR_upper_b]


STEP 8: COMPARE RESULTS
    Compare Historical_VaR with Theoretical_VaR

    CI_width_a = VaR_upper_a - VaR_lower_a
    CI_width_b = VaR_upper_b - VaR_lower_b

    Compare CI_width_a with CI_width_b

    Interpret the effect of parameter uncertainty

END
