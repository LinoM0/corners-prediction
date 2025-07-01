------------------------------------------------------------------------

## Summary

-   **Introduction and Objective**

    -   Predict the total number of corners in football matches using limited available data (league ID, date, home and away team IDs).
    -   Derive betting probabilities (under, exactly, over a given line) and size bets based on model predictions.

-   **Data Preparation and Exploratory Data Analysis (EDA)**

    -   **Data Cleaning and Preparation:**

        -   Converted dates to date objects and IDs to factors.
        -   Derived additional features: year, month, day, total goals, goal difference, total corners, match outcomes.

    -   **Key Insights from EDA:**

        -   13 leagues (most with sufficient observations).
        -   Most leagues contain around 30 teams over the observed period.
        -   Some teams have fewer than 10 matches.
        -   Home team advantage clearly visible in game outcomes.
        -   League-specific effects clearly exist.
        -   Team-specific effects present but weaker.
        -   Home team advantage visible in goals and corners.
        -   Negative correlation between home and away corners observed.
        -   Slight upward trend in total corners over time identified.
        -   Minimal to no correlation between goals statistics and corners, and between corners and match outcomes.

    -   **Conclusions from EDA:**

        -   Goals lack sufficient predictive power for corners to use in an intermediate step for modelling corners; thus, models will primarily rely on league and team effects.

        -   Corners seem well approximated by a Poisson distribution (particularly individual team corners; assuming independence, total corners would also follow Poisson).

        -   However, home and away corners are not independent. An negative correlation is observed which is surprising. With hectic and stalemate games, one would expect a positive correlation between home and away corners. This negative correlation could turn positive once other factors are controlled for, but with this few features this is unlikely (tested it as well with residuals of hierarchical Poisson GLMM and negative correlation persisted)

        -   If the correlation were positive, one could also think about fitting a bivariate Poisson model for home and away corners to account for their dependence, but this was not pursued further due to the negative correlation

        -   Therefore, in terms of modelling the following approach was chosen:

            1.  Fit baseline constant-rate and time-decaying Poisson models to establish a simple benchmark
            2.  Build upon simple Poisson models, by modelling league and team effects as random effects, giving rise to a hierarchical Poisson GLMM
            3.  Evaluate whether a non-linear model could improve upon the linear specifications by fitting a random forest model for predicting the $\lambda$'s of multiple Poisson distributions (depending on the splits of the Random Forest)

-   **Baseline Models**

    -   **Constant-rate Poisson Model**

        -   Assumption: Matches within each league have a constant rate of corners.
        -   Mathematical representation: $\text{Total Corners} \sim \text{Poisson}(\lambda_{\text{league}})$

    -   **Time-decaying Poisson Model**

        -   Matches further in the past receive lower weight in estimation (exponential decay with half-life of two years).
        -   Mathematical representation: $\text{Total Corners} \sim \text{Poisson}(\lambda_{\text{league}}), \quad \text{with weights } w_i = e^{-\frac{\log(2) \cdot \Delta t_i}{\text{half-life}}}$

-   **Hierarchical Generalized Linear Mixed Model (GLMM)**

    -   Extended Poisson model with hierarchical random effects for leagues and teams (league and team random effects assumed standard normal).

    -   Separate models for home and away corners, predictions summed to total corners.

    -   Mathematical representation: $\text{Home Corners} \sim \text{Poisson}(\lambda_{\text{home}}) \quad \text{with} \quad \log(\lambda_{\text{home}}) = \beta_0 + u_{\text{league}} + u_{\text{home-team}} + u_{\text{away-team}}$ $\text{Away Corners} \sim \text{Poisson}(\lambda_{\text{away}}) \quad \text{with} \quad \log(\lambda_{\text{away}}) = \beta_0 + v_{\text{league}} + v_{\text{home-team}} + v_{\text{away-team}}$

    -   Over-dispersion check (variance \> 1.5x mean) revealed under-dispersion (factor \~0.63), confirming Poisson appropriateness (Poisson distribution relies on the mean and variance being approximately equal). Otherwise one would need to use another distribution, such as a negative-binomial distribution

    -   **Evaluation compared to baseline models:**

        -   Evaluated using RMSE, Negative Log-Likelihood (NLL), shrinkage plot and bootstrap confidence interval.
        -   All methods point to the conclusion that the extension to a hierarchical Poisson GLMM model does not improve performance greatly, it was still chosen though due to minor performance gains.

-   **Random Forest Model**

    -   Leveraged league ID, team IDs, and month of match for nonlinear modeling.
    -   Hyperparameters tuned via Bayesian optimization.
    -   Model underperformed relative to GLMM (expected given sparse features and minimal feature interactions).

-   **Model Selection and Predictions**

    -   Hierarchical Poisson GLMM selected based on predictive performance.
    -   Generated predictions for test dataset.

-   **Betting Probabilities and Sizing**

    -   Converted predicted corner counts to betting probabilities (below, at, or above given Asian betting lines).

    -   Mathematical probability determination: $P(\text{under}) = P(X \leq \text{line}), \quad P(\text{exact}) = P(X = \text{line}), \quad P(\text{over}) = P(X > \text{line})$

    -   Calculated expected values and Sharpe ratios for each potential bet: $\text{EV} = P_{\text{win}} \cdot (\text{odds} - 1) - P_{\text{loss}}$ $\text{Sharpe Ratio} = \frac{\text{EV}}{\sqrt{\text{Variance}}} \quad \text{with Variance computed from win/loss probabilities}$

    -   Bet-sizing strategies:

        -   Risk-parity approach ensuring portfolio variance is 1% of total bankroll.
        -   Full bankroll utilization, scaling bets to available capital (selected, using all 341 units).

-   **Conclusion and Practical Implications**

    -   Hierarchical Poisson GLMM effectively balances model complexity and performance given the limited data.
    -   Provides robust, data-driven methodology for accurate predictions and informed betting decisions.

------------------------------------------------------------------------


