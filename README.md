# Reinforcement Learning for Dynamic Ridesharing Pricing

An educational machine-learning and reinforcement-learning project that explores dynamic price selection for a ridesharing platform. A Random Forest estimates the expected historical ride cost from the current market state, and a tabular Q-learning agent selects a discrete price multiplier.

The project is organised as four sequential Jupyter notebooks so that data preparation, supervised modelling, RL training, and final policy evaluation remain separate and reproducible.

## Project objectives

- Clean and prepare the Kaggle Dynamic Pricing Dataset.
- Engineer demand and supply features for ridesharing markets.
- Train a Random Forest Regression model to estimate expected base ride cost.
- Formulate dynamic pricing as a reinforcement-learning problem.
- Train a Q-table agent with epsilon-greedy exploration.
- Compare fixed, rule-based, and RL-based pricing policies.
- Evaluate both gross expected revenue and penalty-adjusted reward.

## Dataset

The project uses the [Dynamic Pricing Dataset on Kaggle](https://www.kaggle.com/datasets/arashnic/dynamic-pricing-dataset/data), published by Möbius under the **CC0: Public Domain** licence.

The local file is `dynamic_pricing.csv` and contains 1,000 observations with the following original fields:

- number of riders and drivers;
- location category;
- customer loyalty status, past rides, and average rating;
- booking time and vehicle type;
- expected ride duration;
- historical ride cost.

## Repository structure

```text
.
├── notebooks/
│   ├── 01_data_preparation.ipynb
│   ├── 02_random_forest_training_evaluation.ipynb
│   ├── 03_q_learning_training.ipynb
│   └── 04_pricing_policy_evaluation.ipynb
├── docs/
│   ├── METHODOLOGY.md
│   ├── REPRODUCIBILITY.md
│   └── RESULTS.md
├── report/
│   └── Reinforcement Learning CA Report.pdf
├── artifacts/                 # Generated locally; ignored by Git
├── .github/workflows/
│   └── notebook-smoke-test.yml
├── dynamic_pricing.csv
├── requirements.txt
├── CONTRIBUTING.md
└── README.md
```

## Notebook workflow

Run the notebooks in numerical order:

1. **Data preparation** — audits the raw data, cleans names and values, engineers features, and creates the train/test files.
2. **Random Forest training and evaluation** — fits the preprocessing pipeline and model, reports regression metrics, and saves environment predictions.
3. **Q-learning training** — constructs discrete states, defines the pricing reward, trains the Q-table, and saves training history.
4. **Policy evaluation** — compares fixed, rule-based, and RL pricing on the same held-out test set and creates the final charts.

Each notebook checks for the outputs required from the previous stage and provides a clear error if they are missing.

## Installation

Python 3.10 or later is recommended.

```bash
git clone <your-repository-url>
cd <repository-folder>
python -m venv .venv
```

Activate the environment:

```powershell
# Windows PowerShell
.venv\Scripts\Activate.ps1
```

```bash
# macOS or Linux
source .venv/bin/activate
```

Install the dependencies and start Jupyter:

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
jupyter lab
```

Open the `notebooks` directory and run notebooks 01–04 in order.

## Train/test configuration

- Training set: **800 rows (80%)**
- Test set: **200 rows (20%)**
- Random seed: **42**
- Target: `historical_cost_of_ride`

The preprocessing pipeline is fitted on training data only. Numerical values use median imputation and standardisation; categorical values use most-frequent imputation and one-hot encoding.

## Reinforcement-learning formulation

- **State:** discretised demand, supply, demand/supply ratio, ride duration, customer history, rating, loyalty, location, booking time, and vehicle type.
- **Actions:** `[0.8, 0.9, 1.0, 1.1, 1.2, 1.3]` price multipliers.
- **Adjusted price:** predicted base cost multiplied by the selected action.
- **Demand response:** an elasticity function reduces retained demand as price increases.
- **Reward:** expected adjusted revenue minus a quadratic excessive-pricing penalty above a multiplier of 1.1.

See [docs/METHODOLOGY.md](docs/METHODOLOGY.md) for the complete design.

## Recorded results

### Random Forest test performance

| Metric | Value |
|---|---:|
| MAE | 55.58 |
| RMSE | 74.43 |
| R² | 0.848 |

### Held-out pricing-policy comparison

| Policy | Total expected revenue | Average reward | Average multiplier |
|---|---:|---:|---:|
| Fixed pricing | 77,760.41 | **388.80** | 1.000 |
| Rule-based pricing | **78,041.03** | 364.24 | 1.183 |
| RL pricing | 77,728.37 | 386.36 | 1.084 |

Rule-based pricing achieved the highest simulated gross revenue. Fixed pricing achieved the highest penalty-adjusted reward, while RL pricing was close to fixed pricing under the specified reward. This is an important result rather than a failure: the preferred policy depends on how revenue, demand retention, and excessive pricing are valued.

See [docs/RESULTS.md](docs/RESULTS.md) for interpretation and limitations.

## Reproducing the full pipeline from the command line

After installing dependencies, the notebooks can be executed without opening JupyterLab:

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/01_data_preparation.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/02_random_forest_training_evaluation.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/03_q_learning_training.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/04_pricing_policy_evaluation.ipynb
```

Detailed reproduction instructions are available in [docs/REPRODUCIBILITY.md](docs/REPRODUCIBILITY.md).

## Important limitations

- The dataset is small and represents historical market snapshots rather than a live sequential environment.
- Retained demand is simulated using an assumed price-elasticity function, not estimated from observed customer conversion behaviour.
- The Random Forest predicts historical cost, which is used as a proxy for expected base revenue.
- The Q-table uses coarse state bins and a fallback action for unseen test states.
- Results are based on one train/test split and one random seed.
- A production system would require causal experimentation, uncertainty analysis, price caps, fairness testing, legal review, and customer/driver welfare constraints.

This repository is intended for academic demonstration and should not be used directly for real-world pricing decisions.

## Documentation

- [Methodology](docs/METHODOLOGY.md)
- [Reproducibility guide](docs/REPRODUCIBILITY.md)
- [Results and interpretation](docs/RESULTS.md)
- [Complete project report](report/Reinforcement%20Learning%20CA%20Report.pdf)
- [Contributing guide](CONTRIBUTING.md)

## Publishing the repository to GitHub

The project directory is prepared for Git but has not been connected to a GitHub remote. After creating an empty repository on GitHub, run the following from this directory:

```bash
git init
git add .
git commit -m "Initial dynamic pricing RL project"
git branch -M main
git remote add origin https://github.com/<your-username>/<repository-name>.git
git push -u origin main
```

Run `git status --ignored` before the first push if you want to confirm that `.idea/`, local environments, notebook checkpoints, and generated `artifacts/` files are excluded.
