# Contributing

This is an academic project, but reproducibility improvements, documentation corrections, and clearly justified modelling experiments are welcome.

## Development setup

1. Create and activate a Python virtual environment.
2. Install `requirements.txt`.
3. Run the notebooks in numerical order.
4. Confirm all notebook cells execute without errors.

## Making changes

- Keep each notebook focused on its existing responsibility.
- Do not introduce target leakage from `historical_cost_of_ride` into model features or RL states.
- Fit preprocessing and bin boundaries using training data only.
- Use random seed 42 unless an experiment explicitly studies multiple seeds.
- Document changes to reward assumptions or price elasticity because these directly change policy rankings.
- Do not commit generated files from `artifacts/` or IDE-specific metadata.

## Before submitting changes

Run the full sequence:

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/01_data_preparation.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/02_random_forest_training_evaluation.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/03_q_learning_training.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/04_pricing_policy_evaluation.ipynb
```

Then check that:

- no cell contains an exception;
- model metrics and policy summaries are generated;
- README result values remain accurate;
- new assumptions and limitations are documented.
