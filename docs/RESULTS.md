# Results and Interpretation

## Random Forest model

The held-out test set contains 200 observations.

| Metric | Result |
|---|---:|
| Mean absolute error | 55.58 |
| Root mean squared error | 74.43 |
| R² | 0.848 |

The R² result indicates that the model explains a substantial proportion of historical ride-cost variation in this split. RMSE is larger than MAE, showing that some observations have comparatively large prediction errors.

These metrics describe prediction of historical ride cost. They do not validate the simulated customer demand response used by the RL environment.

## Pricing-policy comparison

| Policy | Total expected revenue | Average reward | Average multiplier |
|---|---:|---:|---:|
| Fixed pricing | 77,760.41 | **388.80** | 1.000 |
| Rule-based pricing | **78,041.03** | 364.24 | 1.183 |
| RL pricing | 77,728.37 | 386.36 | 1.084 |

## Interpretation

The rule-based policy produces the highest gross expected revenue, exceeding fixed pricing by approximately 280.62 in the simulation. It also selects the largest average multiplier and receives a substantially lower reward because the reward includes an excessive-pricing penalty.

Fixed pricing produces the highest average penalty-adjusted reward. RL pricing is close to fixed pricing but does not outperform it under the current state abstraction, elasticity assumptions, penalty strength, and unseen-state fallback.

The comparison demonstrates why the optimisation objective must be stated explicitly. “Best” means different things for gross revenue and for a reward that also reflects demand loss and excessive pricing.

## What can be concluded

- The Random Forest is a useful predictive proxy for base ride cost on this dataset and split.
- Pricing-policy rankings change depending on whether gross revenue or penalised reward is optimised.
- A simple rule can remain competitive with tabular RL when data is limited and the environment is simulated.
- Reporting a baseline that beats RL is scientifically preferable to claiming an unsupported RL improvement.

## What cannot be concluded

- The results do not prove that any policy will increase real ridesharing revenue.
- The simulated retained-demand function is not a causal estimate of customer behaviour.
- The model does not establish that the chosen prices are fair, lawful, or welfare-improving.
- A single split and seed do not quantify uncertainty.

## Recommended extensions

1. Estimate price elasticity from real booking or conversion experiments.
2. Use repeated splits or cross-validation for the supervised model.
3. Evaluate multiple RL seeds and report confidence intervals.
4. Tune state bins, learning rate, episode count, and penalty parameters on validation data rather than the test set.
5. Add price caps, fairness metrics, and customer/driver welfare constraints.
6. Compare Q-learning with contextual bandits, which better match independent market snapshots.
7. Test the policy in an online simulator before any controlled field experiment.
