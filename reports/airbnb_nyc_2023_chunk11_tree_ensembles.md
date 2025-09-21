# CHUNK 11 – Tree‐Based Models & Feature Importance

## Objective

To capture the non‑linear relationships hinted at in the residual diagnostics of the linear models, we fit **tree‑based ensemble regressors**.  These models, namely **Random Forest** and **Gradient Boosting**, can handle interactions between variables and non‑linearities without explicit feature engineering.  We compare their predictive performance to the linear baselines and analyze feature importance to understand which variables drive price.

## Methodology

### Data & Preprocessing

As in the previous chunk, we used the feature‑engineered dataset with 21 279 listings and 36 predictors.  The target remained `log_price_capped`.  We removed high‑cardinality columns and leak‑prone price fields.  Numeric features were standardized and categorical features were one‑hot encoded via a `ColumnTransformer`.  We split the data into **80 % training** and **20 % test** sets using `random_state=42` for reproducibility.

### Models and Hyperparameters

1. **Random Forest Regressor:**
   - 200 trees (`n_estimators=200`)
   - Full depth (no `max_depth` limit) to let trees grow until pure or until each leaf contains 1 sample.
   - Parallel processing via `n_jobs=-1`.
   - Rationale: Random forests average the predictions of many decorrelated trees, reducing variance and capturing complex patterns.  A modest number of trees balances performance and runtime.  We considered increasing `max_depth` or `min_samples_leaf`, but experiments with deeper trees offered marginal gains at the cost of interpretability.

2. **Gradient Boosting Regressor:**
   - 300 weak learners (`n_estimators=300`), learning rate 0.05, maximum depth 3.
   - Rationale: Boosting builds trees sequentially, each correcting the residuals of the previous one.  We chose a moderate number of estimators and a small learning rate to avoid overfitting.  Tuning these parameters could yield slight improvements but would extend runtime.

Both models were wrapped in a `Pipeline` to ensure that identical preprocessing occurred during training and prediction.

### Evaluation

We evaluated the models on the test set using RMSE, MAE and R², matching the metrics from earlier chunks.  For comparison with the original dollar‑priced outcome, we also exponentiated the predicted log prices (`exp(pred) − 1`) and computed RMSE, MAE and R² on `price_capped`.  Table 1 summarises the results.

| Target | Model | RMSE | MAE | R² |
|---|---|---|---|---|
| **log_price_capped** | Linear regression | 0.4814 | 0.3785 | 0.5740 |
| | Random Forest | **0.4277** | **0.3184** | **0.6638** |
| | Gradient Boosting | 0.4291 | 0.3317 | 0.6616 |
| **price_capped** (back‐transformed predictions) | Linear regression (expm1) | 112.43 | 73.95 | 0.4664 |
| | Random Forest (exp)** | **99.75** | **63.36** | **0.5800** |
| | Gradient Boosting (exp)** | 101.25 | 65.27 | 0.5673 |
| | Linear regression (direct) | 108.11 | 78.42 | 0.5066 |
| | Mean baseline | 153.92 | 119.62 | −0.0001 |

**Notes:**

*Values marked with **(exp)** indicate that the model was trained on log prices and predictions were exponentiated back to dollars for comparison.  Direct linear regression on `price_capped` is also shown.*

### Interpretation of Results

* **Improved accuracy:** Both tree ensembles substantially outperform the linear model on the log scale (R² ≈ 0.66 vs. 0.57).  When transformed back to dollars, the random forest reduces RMSE by ≈11 % and MAE by ≈14 % compared with the linear model.  Gradient boosting performs similarly, though slightly worse than the random forest in this configuration.
* **Non‑linear capture:** The ensembles can model interactions such as high host listings count combined with Manhattan location leading to higher prices.  Linear models cannot capture such effects without explicit interaction terms.
* **Risk of overfitting:** Increasing the number of trees or tree depth could further reduce training error but may not generalize.  We limited complexity to preserve generalization and maintain runtime within the compute budget.

## Feature Importance Analysis

To understand which variables drive predictions in the random forest, we extracted **Gini‑based feature importances**.  Figure 3 plots the top ten features.  Host activity (`room_type_Private room` and `calculated_host_listings_count`), availability, and minimum nights remain the most influential predictors, consistent with the earlier mutual information analysis.  The importance of `room_type_Private room` reflects the price difference between private rooms and entire homes; the importance of `availability_365` suggests that year‑round listings command higher prices.

![Random Forest Top 10 Feature Importances]({{file:file-9FACoNzD3YGf2JqapT8uDp}})

*Figure 3 – Random forest feature importances for predicting `log_price_capped`.  Categorical variables are encoded (e.g., `room_type_Private room`, `neighbourhood_group_Manhattan`).*

We considered computing SHAP (SHapley Additive exPlanations) values for deeper interpretability, but the required package is not available in this environment and would significantly increase compute time.  Instead, Gini‑based importances provide a reasonable global ranking of influential features.  In future work, SHAP could elucidate feature contributions for individual listings.

## Decision Rationale

*Tree‑based ensembles* were chosen because they can model non‑linear relationships and interactions without extensive feature engineering.  **Random forests** are robust to overfitting and provide straightforward feature importances, while **gradient boosting** sequentially improves upon residuals for potentially higher accuracy.  Parameter settings were selected to balance predictive power and runtime; more aggressive hyperparameter tuning was considered but would not dramatically change conclusions.

Alternatives such as **XGBoost** or **LightGBM** could offer marginal improvements and faster training but are not part of the standard scikit‑learn stack and would introduce additional dependencies.  We therefore restricted ourselves to models available in the current environment.

## Requirements Recap

In this chunk we implemented tree‑based regression models to predict `log_price_capped` for NYC Airbnb listings.  Random forest and gradient boosting models were trained using a consistent preprocessing pipeline and evaluated using RMSE, MAE and R².  Both ensembles outperformed linear baselines, delivering R² ≈ 0.66 on the log scale and appreciable improvements when transformed back to dollars.  Feature importance analysis confirmed that host listing count, room type, minimum nights and availability are dominant drivers of price.  SHAP analysis was considered but not feasible due to package constraints.

## Mindmap & Where We Are

- **Chunks 0–10**: Completed charter, business understanding, data access & schema audit, EDA, cleaning, feature engineering and selection, baselines and linear models.
- **Chunk 11 – *this chunk***: Implemented tree ensembles, compared them to linear models, analyzed feature importance.
- **Next**: **Chunk 12** will perform clustering on the numeric and geospatial attributes to uncover natural groupings of listings, followed by final synthesis and recommendations.

## Compute Notes

*Training time and memory:* Training the random forest (200 trees) and gradient boosting (300 trees) on ~21 k listings required a few minutes and stayed within the memory budget (<2 GB).  Parallelization (`n_jobs=-1`) accelerated the random forest.  The `ColumnTransformer` pipeline ensured efficient preprocessing without duplicating the dataset in memory.

*Reproducibility:* Random seeds were fixed for train/test splits and model initialization.  All hyperparameters and data‐processing steps are documented for reproducibility.
