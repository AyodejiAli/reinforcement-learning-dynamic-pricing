# Methodology

## 1. Problem definition

The project studies whether a tabular reinforcement-learning policy can choose ridesharing price multipliers from observed market conditions while balancing revenue with a penalty for excessive pricing.

The implementation has two modelling layers:

1. a Random Forest Regression model estimates expected historical/base ride cost from a market state;
2. a Q-learning agent applies a price multiplier to that estimate inside a simulated demand-response environment.

## 2. Data preparation

The raw dataset contains 1,000 rows. Column names are converted to `snake_case`, exact duplicates are removed, categorical whitespace is cleaned, and invalid infinite values are converted to missing values.

Engineered features include:

```text
demand_supply_ratio = number_of_riders / max(number_of_drivers, 1)
riders_per_minute   = number_of_riders / max(expected_ride_duration, 1)
experienced_customer = 1 when past rides are at least the dataset median
```

The data is split once using an 80/20 random split with seed 42. The result is 800 training rows and 200 testing rows.

## 3. Supervised environment model

The target is `historical_cost_of_ride`. Input features are:

- number of riders and drivers;
- demand/supply ratio and riders per minute;
- expected ride duration;
- number of past rides, rating, and experienced-customer flag;
- location, customer loyalty, booking time, and vehicle type.

The preprocessing pipeline performs:

- median imputation and standardisation for numerical features;
- most-frequent imputation and one-hot encoding for categorical features.

All preprocessing is fitted on training data only. The main estimator is a Random Forest Regressor with 300 trees, a minimum leaf size of 2, seed 42, and parallel execution.

Random Forest was selected because it handles nonlinear relationships and feature interactions in mixed tabular data without requiring strong distributional assumptions.

## 4. RL state and actions

The conceptual state includes current demand, supply, demand pressure, ride duration, customer history/rating/loyalty, location, time, and vehicle type.

Continuous fields are converted to quantile-based bins learned from training data:

- riders: 2 bins;
- drivers: 2 bins;
- demand/supply ratio: 3 bins;
- ride duration: 2 bins;
- past rides: 2 bins;
- average rating: 2 bins.

Categorical fields are mapped to integer codes. This compressed tuple becomes the Q-table key.

The action space is:

```python
[0.8, 0.9, 1.0, 1.1, 1.2, 1.3]
```

## 5. Pricing environment and reward

For predicted base cost `C` and action multiplier `m`:

```text
adjusted_price = C × m
```

Price sensitivity is simulated rather than learned from the dataset. Demand pressure is clipped between 0.5 and 3.0, and local price elasticity is:

```text
local_elasticity = 1.35 / sqrt(demand_supply_ratio)
```

Retained demand is:

```text
retained_demand = clip(exp(-local_elasticity × (m - 1)), 0.45, 1.10)
```

Expected revenue and the excessive-price penalty are:

```text
expected_revenue = adjusted_price × retained_demand
excess = max(0, m - 1.10)
penalty = C × 8.0 × excess²
reward = expected_revenue - penalty
```

Gross expected revenue and penalised reward are reported separately so the financial output is not confused with the policy objective.

## 6. Q-learning

The agent uses:

- learning rate: 0.12;
- discount factor: 0.90;
- initial epsilon: 1.00;
- epsilon decay: 0.97 per episode;
- minimum epsilon: 0.05;
- training episodes: 180.

Each dataset row is an independent market snapshot and is treated as a terminal contextual decision. Consequently, the future-state component is zero during the current experiment even though the standard discount-factor implementation remains available for future sequential environments.

For test states absent from the training Q-table, the policy selects the action with the highest mean Q-value across known states.

## 7. Baselines

### Fixed policy

Always selects multiplier 1.0.

### Rule-based policy

- ratio above 1.50: multiplier 1.2;
- ratio above 1.10: multiplier 1.1;
- ratio below 0.80: multiplier 0.9;
- otherwise: multiplier 1.0.

### RL policy

Selects the action with the largest Q-value for the state, using the learned fallback for unseen states.

## 8. Evaluation

Random Forest performance is measured using MAE, RMSE, and R². Pricing policies are compared on the same 200 held-out market states using:

- total expected revenue;
- average penalty-adjusted reward;
- average selected multiplier;
- average retained demand.

This common test set makes the policy comparison internally consistent, but it does not replace an online controlled experiment.
