# CHUNK 9 – Feature Selection (NYC Airbnb 2023)

## Summary

After feature engineering we obtained a modelling table with **21 279 listings** and **26 columns**.  This chunk evaluates which predictors are most informative for the target `log_price_capped`.  The goal is to reduce dimensionality, avoid multicollinearity and retain variables that offer predictive power or business insight.  We use three complementary techniques—Pearson correlation with the target, *mutual information* (MI) and *random‑forest feature importance*—and triangulate a final candidate set of features for modelling.

## Approach and Decision Rationale

1. **Removing high‐cardinality columns and leakage‑prone features.**  The original dataset contains a granular `neighbourhood` column with hundreds of categories.  One‑hot encoding this column would explode the dimensionality and hinder interpretability; moreover, many neighbourhoods have few observations.  Instead, we rely on the coarser `neighbourhood_group` and drop `neighbourhood`.  We also remove raw price variables (`price`, `price_capped`, `log_price`) from the feature matrix to avoid leakage.  The target is `log_price_capped`, chosen in the previous chunk because log transformation stabilises heteroskedasticity and downweights extreme prices【559312119136520†L120-L124】【559312119136520†L169-L177】.
2. **Encoding categorical variables.**  Categorical predictors (`room_type`, `neighbourhood_group`, and the engineered bins) are one‑hot encoded with `drop_first=True` to prevent the dummy‑variable trap.  This produces a feature matrix with **36 columns**, a manageable size for both linear and tree‑based models.
3. **Quantifying relationships.**  We compute:
   - **Pearson correlation** between each numeric feature and the target to capture linear dependencies.
   - **Mutual information** (MI) using `mutual_info_regression` from scikit‑learn, capturing both linear and non‑linear associations.  MI is non‑negative and higher values indicate greater information about the target.
   - **Random‑forest importance** via a 150‑tree ensemble.  Trees handle non‑linearities and interactions, so features with high importance contribute to splitting decisions.
4. **Selecting candidate features.**  We rank features by each metric and look for variables consistently appearing near the top.  Features with negligible importance across metrics are earmarked for removal.  We also watch for highly collinear pairs (e.g., raw and log‑transformed counts).  For subsequent modelling we retain variables that are theoretically relevant (host activity, minimum nights, availability, location and review activity) and show evidence of predictive power.

### Alternatives Considered

- *Automated dimensionality reduction* (e.g., Principal Component Analysis) was considered but rejected: PCA mixes variables into latent components that are difficult to interpret for business recommendations.
- *Recursive feature elimination* (RFE) could iteratively eliminate weak predictors, but on a 21k‑row dataset with 30–40 predictors it provides limited benefit over the simpler filter methods.  We use tree‑based feature importance instead as a wrapper method.

## Findings

The table below summarises the top features by correlation, mutual information and random‑forest importance.  Only the highest‑ranked predictors are shown; see the code for the full list.

| Feature | Correlation (with log_price_capped) | MI Score | Random‑Forest Importance |
|---|---|---|---|
| `log_host_listings_count` | 0.15 | **0.478** | 0.085 |
| `calculated_host_listings_count` | **0.27** | **0.473** | 0.079 |
| `room_type_Private room` | – | 0.190 | **0.295** |
| `minimum_nights` | −0.085 | 0.134 | 0.045 |
| `log_minimum_nights` | −0.200 | 0.131 | 0.043 |
| `minimum_nights_capped` | −0.160 | 0.123 | 0.046 |
| `availability_365` | 0.039 | 0.106 | 0.097 |
| `neighbourhood_group_Manhattan` | – | 0.089 | 0.043 |
| `days_since_last_review` | −0.011 | 0.072 | 0.036 |
| `reviews_per_month` | 0.035 | 0.069 | 0.027 |
| `log_reviews_per_month` | 0.051 | 0.067 | 0.027 |
| `log_days_since_last_review` | −0.019 | 0.066 | 0.035 |
| `log_number_of_reviews` | −0.074 | 0.063 | 0.021 |
| `min_nights_gt_30` | 0.250 | – | – |

**Key observations:**

* Host activity variables (`log_host_listings_count` and `calculated_host_listings_count`) are highly informative across all metrics.  A higher number of listings owned by the host is strongly associated with higher nightly prices.  This may reflect professional hosts with better marketing or higher‑end properties.
* Room type matters: the indicator for `room_type_Private room` has a **29 % feature importance** in the random‑forest model and a high MI score.  Private rooms are typically cheaper, so when the dummy is 1 the log price decreases relative to the omitted category (`Entire home/apt`).
* Minimum nights and its transformations show negative correlations, indicating that listings with longer minimum stays tend to charge lower nightly rates.  The capped and log versions appear among the top features, but they are highly correlated with each other; we will evaluate whether to keep both or select a single representation to avoid multicollinearity.
* Availability (`availability_365`) has modest correlation but high importance in tree models; listings available for more days often command higher prices.
* The `neighbourhood_group_Manhattan` dummy emerges as an influential location feature—aligning with expectations that Manhattan properties are pricier.
* Temporal and review‑frequency variables (`days_since_last_review`, `reviews_per_month`, `log_reviews_per_month`, `log_days_since_last_review`) contribute moderate MI and importance, suggesting that active and recently reviewed listings may price differently.
* Variables like `min_nights_gt_30` have high correlation but low MI and are not chosen by the random forest, highlighting limitations of relying solely on linear measures.

### Proposed Feature Set

Based on these findings and domain knowledge, we propose the following predictors for modelling:

1. **Host scale**: `log_host_listings_count`, `calculated_host_listings_count`, and the engineered `host_listings_bin` dummies capture the host’s portfolio size and are strong predictors of price.
2. **Room type**: one‑hot encoding of `room_type` (e.g., `room_type_Private room`, `room_type_Shared room`) will be kept because of their high importance.
3. **Minimum nights**: we will test both `log_minimum_nights` and `minimum_nights_capped`.  If multicollinearity becomes problematic in linear models we will retain only the log or capped version.
4. **Availability**: `availability_365` expresses the number of days the listing is available, which shows strong importance in the tree model.
5. **Neighbourhood group**: dummies for `neighbourhood_group` (e.g., `neighbourhood_group_Manhattan`) supply location context without the high‑cardinality `neighbourhood` column.  Inside Airbnb notes that coordinates are anonymised by up to 150 m【788864212863535†L40-L44】, so neighbourhood group is a reliable high‑level location indicator.
6. **Review activity**: `reviews_per_month`, `log_reviews_per_month`, `log_days_since_last_review` and `days_since_last_review` gauge the listing’s popularity and recency of engagement.  We will evaluate whether both raw and log forms are needed.
7. **Other engineered bins** (review count bins, host listings bins, availability bins, recency bins) can provide non‑linear thresholds and are low‑cardinality; we include them initially for tree‑based models.

Variables demonstrating little predictive power (low MI and importance) will be dropped in later modelling.  We will monitor multicollinearity using variance inflation factors (VIF) for linear models and prune redundant variables accordingly.

## Requirements Recap

In this chunk we selected features for modelling the log‑capped price of NYC Airbnb listings.  We used correlation, mutual information and random‑forest importance to evaluate 36 engineered predictors, dropped high‑cardinality or leakage‑prone variables, and assembled a candidate feature set focusing on host scale, room type, minimum nights, availability, location and review activity.  Decisions were justified by comparing alternative methods and acknowledging trade‑offs.

## Mindmap & Where We Are

- **Project Charter (Chunk 0)**: defined objectives, dataset scope and CRISP‑DM roadmap.
- **Business/Data Understanding (Chunk 1)**: summarised NYC Airbnb market and dataset context.
- **Data Access & Schema Audit (Chunk 2)**: safely loaded data, inspected schema and missingness.【788864212863535†L40-L44】
- **EDA (Chunks 3–5)**: explored distributions, bivariate/temporal patterns and geospatial patterns.
- **Cleaning & Pre‑processing (Chunk 6)**: handled missing values, capped/extensively transformed features, created `log_price_capped` target.
- **Outlier Analysis (Chunk 7)**: identified and treated extreme values.
- **Feature Engineering (Chunk 8)**: created log transforms, caps, bins for hosts, reviews and availability, saved `listings_feature_engineered.csv`.
- **Feature Selection (Chunk 9 – *this chunk*)**: evaluated predictors using correlation, mutual information and random‑forest importance; proposed final feature set.
- **Next**: proceed to **Chunk 10** (Modelling—Baselines & Linear Models), using the selected features.

## Compute Notes

* **Resource management**: The feature‑engineered dataset (~4.6 MB) fits easily within the ~12 GB memory constraint.  Loading and encoding the 21 k rows is inexpensive.  We set `n_jobs=-1` for RandomForestRegressor to parallelise across CPU cores within the compute budget.
* **Reproducibility**: Random seeds (e.g., 42) were fixed for mutual information and train‑test splitting.  We used `drop_first` in dummy encoding to avoid redundant columns.
* **Downstream modelling**: The proposed feature set keeps the feature matrix manageable (≈30–40 features), ensuring that linear and tree‑based models run in under ~5–8 minutes with the available compute.
