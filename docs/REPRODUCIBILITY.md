# Reproducibility Guide

## Prerequisites

- Python 3.10 or later;
- the repository's `dynamic_pricing.csv` file;
- dependencies from `requirements.txt`.

## Environment setup

From the repository root:

```bash
python -m venv .venv
```

Activate the environment and install dependencies:

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

## Required execution order

The notebooks exchange files through the local `artifacts/` directory and must be executed in this order:

1. `notebooks/01_data_preparation.ipynb`
2. `notebooks/02_random_forest_training_evaluation.ipynb`
3. `notebooks/03_q_learning_training.ipynb`
4. `notebooks/04_pricing_policy_evaluation.ipynb`

The notebooks support being launched either from the repository root or from the `notebooks/` directory.

## Command-line execution

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/01_data_preparation.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/02_random_forest_training_evaluation.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/03_q_learning_training.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/04_pricing_policy_evaluation.ipynb
```

## Generated artifacts

Notebook 01 generates cleaned data and the train/test split. Notebook 02 saves the preprocessing/model pipeline, regression metrics, and environment cost predictions. Notebook 03 saves the Q-table configuration and reward history. Notebook 04 saves row-level policy outcomes and the summary comparison.

Generated artifacts are intentionally excluded from Git because they can be recreated. The directory is retained using `artifacts/.gitkeep`.

Expected files include:

```text
artifacts/
├── cleaned_dynamic_pricing.csv
├── data_manifest.json
├── train_features.csv
├── train_target.csv
├── test_features.csv
├── test_target.csv
├── random_forest_revenue_model.joblib
├── model_metrics.csv
├── train_environment_costs.csv
├── test_environment_costs.csv
├── q_learning_artifacts.joblib
├── training_rewards.csv
├── policy_level_results.csv
└── policy_comparison.csv
```

## Determinism

Python, NumPy, the train/test split, and Random Forest all use seed 42. Q-learning uses the same seeded NumPy random generator. Results should be reproducible with compatible package versions, although minor floating-point differences can occur across platforms and library releases.

## Verification checklist

- All notebook code cells have execution counts.
- No notebook output contains an exception.
- The data manifest reports 800 training and 200 testing rows.
- Random Forest R² is approximately 0.848.
- The final policy comparison contains three rows.
- All generated prices and multipliers are finite.

## Troubleshooting

### A notebook reports missing artifacts

Run all earlier notebooks in numerical order. Do not begin with notebook 03 or 04 in a clean checkout.

### The CSV cannot be found

Confirm that `dynamic_pricing.csv` is located in the repository root and that Jupyter was launched from either the root or `notebooks/` directory.

### `OneHotEncoder` rejects `sparse_output`

Install the versions from `requirements.txt`. Older scikit-learn releases used the deprecated `sparse` parameter.

### Windows blocks Jupyter kernel connection files

Run Jupyter from a normal user terminal with permission to write to the user runtime directory. Corporate sandbox policies may require an approved runtime folder.
