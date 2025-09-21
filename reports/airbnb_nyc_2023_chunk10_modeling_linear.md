# CHUNK 10 – Baseline & Linear Modeling

## Objective

The first modeling step uses linear techniques to benchmark performance and set a baseline for the more flexible models that follow.  We compare simple baselines (predicting the mean or median price) against ordinary least squares (OLS), Ridge and Lasso regressions.  We evaluate models on both the capped nightly price (`price_capped`) and its log‐transformed counterpart (`log_price_capped`).

## Methodology

### Data Preparation

- **Feature set:**  All engineered predictors from the previous chunk were used, excluding `neighbourhood` (high cardinality) and the target and raw price columns.  Numeric features were standardized, and categorical variables (`room_type`, `neighbourhood_group`, and the engineered bins) were one‑hot encoded with a drop‑first policy to prevent redundancy.  A `ColumnTransformer` and `Pipeline` ensured identical transformations on training and test data.
- **Train/test split:**  The data was split into **80 % training** and **20 % test** sets using a fixed random seed (42) for reproducibility.  This hold‑out strategy preserves an unbiased assessment of predictive performance【788864212863535†L40-L44】.
- **Baselines:**  As naive references, we computed predictions equal to the mean and median of the training target.  These baselines provide a low bar that any useful model should surpass.
- **Models:**  We fitted ordinary linear regression, Ridge regression (`α=1.0`) and Lasso regression (`α=0.001`).  Ridge and Lasso penalize large coefficients to mitigate multicollinearity and overfitting; the penalty weight was selected heuristically as a mild regularizer.  A more exhaustive hyperparameter search could marginally improve performance but was deferred to save compute.

### Evaluation Metrics

Each model was assessed on the test set using:

1. **Root Mean Squared Error (RMSE)** – penalizes larger errors more heavily and is in the same units as the target.
2. **Mean Absolute Error (MAE)** – more robust to outliers; smaller values indicate better fit.
3. **R² (coefficient of determination)** – represents the proportion of variance explained; values closer to 1 indicate better explanatory power.

## Results

The table below summarizes model performance.  Lower RMSE/MAE and higher R² values are better.

| Target | Model | RMSE | MAE | R² |
|---|---|---|---|---|
| **price_capped** | Mean baseline | 153.92 | 119.62 | −0.0001 |
| | Median baseline | 162.81 | 112.72 | −0.1189 |
| | Linear regression | **108.11** | **78.42** | **0.5066** |
| | Ridge | 108.11 | 78.42 | 0.5066 |
| | Lasso | 108.11 | 78.42 | 0.5066 |
| **log_price_capped** | Mean baseline | 0.7377 | 0.6108 | −0.0001 |
| | Median baseline | 0.7385 | 0.6108 | −0.0022 |
| | Linear regression | **0.4814** | **0.3785** | **0.5740** |
| | Ridge | 0.4814 | 0.3785 | 0.5740 |
| | Lasso | 0.4837 | 0.3809 | 0.5700 |

### Interpretation

* **Baselines**: Predicting a constant value performs poorly – R² is near zero or negative, meaning the naive models explain little or none of the variance in nightly price.  This underscores the need for modelling.
* **OLS vs. Regularized models**: Ordinary linear regression achieved an R² of about **0.51** on price and **0.57** on log price.  Ridge and Lasso performed almost identically, indicating that the engineered features are not highly collinear; small penalties had little effect.
* **Log transformation improves fit**: Modelling log price yields a higher R² than modelling raw price (0.574 vs. 0.507) and smaller RMSE/MAE on the log scale.  However, when exponentiating predictions back to dollars (using `exp(pred) – 1`), the resulting metrics (RMSE≈112.4, MAE≈74.0, R²≈0.47) were slightly worse than training directly on price.  This reflects the bias introduced when transforming back from log space.
* **Residual diagnostics**: Figure 1 shows predictions versus actual values on the log scale for the OLS model.  Points broadly follow the 45‑degree line, but there is noticeable spread for high‑priced listings.  Figure 2 plots residuals against predicted values; variance increases slightly for large predictions, hinting at heteroskedasticity not fully captured by the linear model.  These patterns motivate the exploration of non‑linear models in the next chunk.

![Predicted vs Actual (log-scale)]({{file:file-Aa1hJXLqm2K9KUX6at7uh3}})

*Figure 1 – Scatter of predicted vs. actual log prices for the linear regression model.*

![Residuals vs Predicted (log-scale)]({{file:file-TQrq9wM1woXng4x58fStcR}})

*Figure 2 – Residuals versus predicted log prices indicate minor heteroskedasticity.*

## Decision Rationale

Why choose these models and metrics:

* **Baselines** provide a fundamental benchmark and help contextualize improvements.  If a sophisticated model cannot beat the mean/median, it is uninformative.
* **Linear regression** is interpretable—coefficients directly indicate how features change the log price.  Even if it cannot capture non‑linearities, it serves as a baseline for more complex models.
* **Ridge and Lasso** mitigate potential multicollinearity by shrinking coefficients.  While our engineered feature set exhibits limited collinearity, testing them confirms that penalization does not degrade performance and may help generalization in future scenarios.
* **Evaluation on both raw and log targets** demonstrates the trade‑offs of transformation.  Log scaling reduces skewness and yields higher R² in log space【559312119136520†L120-L124】【559312119136520†L169-L177】, but back‑transforming predictions introduces bias; thus, both perspectives are assessed.

Alternatives considered:  We contemplated cross‑validated polynomial regression or interaction terms but rejected them due to the risk of overfitting and increased complexity.  Instead, we will turn to non‑linear tree‑based models (Random Forests and Gradient Boosting) in the next chunk, which can capture complex relationships without manual feature interactions.

## Requirements Recap

This chunk built baseline and linear regression models on the engineered NYC Airbnb dataset.  Using an 80/20 train/test split and a preprocessing pipeline, we compared mean/median baselines with OLS, Ridge and Lasso models on both raw and log‑capped nightly price.  Linear models significantly outperformed baselines and showed better fit on the log scale, though back‑transformed predictions slightly underperformed the direct price model.  Residual diagnostics hinted at non‑linear patterns, motivating tree‑based models in the next step.

## Mindmap & Where We Are

- **Chunks 0–9**: Completed project charter, business understanding, data access and schema audit, exploratory analyses, cleaning, feature engineering and selection.
- **Chunk 10 – *this chunk***: Built baselines and linear models; evaluated performance; identified the need for more flexible methods.
- **Next**: **Chunk 11** will implement tree‑based models (Random Forests and Gradient Boosting), compare them to linear models and investigate feature importances via SHAP values on a sample.

## Compute Notes

* **Efficient preprocessing**: Using `ColumnTransformer` and `Pipeline` allowed us to standardize numerical features and one‑hot encode categoricals in one step, keeping code modular and avoiding data leakage.
* **Runtime and memory**: Training all linear models on ~21 k rows and 36 features took under a minute and consumed minimal memory (<150 MB).  This fits comfortably within the ~12 GB RAM budget.
* **Reproducibility**: A fixed random seed ensured consistent train/test splits.  Hyperparameters (e.g., `α` for Ridge/Lasso) were documented and can be tuned further if needed.
