# CHUNK 8 – Feature Engineering

## Objectives

Feature engineering transforms raw data into informative predictors that improve model performance.  Building on the cleaned dataset, we create new variables capturing non‑linear relationships, host behaviour, review dynamics and temporal activity.  We also apply logarithmic transformations to reduce skewness and facilitate linear modelling.

## Derived Numerical Features

### Winsorised price and logarithms

We retained the original `price` and `log_price` variables, but also included a **winsorised price** (`price_capped`) capped at the 95th percentile (about $615) to reduce the influence of extreme values (see Chunk 7).  Correspondingly, `log_price_capped` is the natural logarithm of `price_capped + 1`.  Modelling both raw and capped prices enables us to compare sensitivity to outliers.

### Log‑transformed predictors

High skewness in several numeric features motivated log transformations.  For each variable x, we computed `log_x = log1p(x)` to compress large values and stabilise variance:

* `log_minimum_nights` – Based on `minimum_nights` capped at 90 nights to remove extreme values.
* `log_number_of_reviews` – Compresses large review counts.
* `log_reviews_per_month` – Handles skew in monthly review rates.
* `log_host_listings_count` – Reflects host portfolio size, mitigating the influence of professional hosts with hundreds of listings.
* `log_number_of_reviews_ltm` – Accounts for recent review activity.
* `log_days_since_last_review` – Captures recency on a log scale.

These transforms maintain the order of observations and are particularly beneficial for linear models.

## Categorical Feature Engineering

### Binning continuous variables

To capture non‑linear effects and simplify interpretation, we discretised several variables into categories:

* **`min_nights_gt_30`** – Binary flag indicating long‑stay listings (1 if minimum stay > 30 nights, else 0).  Roughly 27 % of listings require more than 30 nights (see Chunk 7) and typically have lower nightly rates.
* **`review_count_bin`** – Categorises `number_of_reviews` into bands: 0, 1–5, 6–20, 21–50 and >50 reviews.  The distribution is fairly balanced across categories, with the highest share (about 30 %) having no reviews.  Full counts:
  
  |Reviews|Count|
  |---|---|
  |0|6319|
  |1–5|4315|
  |6–20|3556|
  |21–50|2708|
  |51+|4381|

* **`host_listings_bin`** – Groups `calculated_host_listings_count` into 1, 2–5, 6–20 and 21+ listings.  This distinguishes occasional hosts from professionals.  Counts:
  
  |Host listing count|Count|
  |---|---|
  |1|7592|
  |2–5|5507|
  |6–20|3028|
  |21+|5152|

* **`availability_bin`** – Buckets `availability_365` into 0 days, 1–100, 101–200, 201–300 and 301–365 days.  Highly available listings (301–365 days) are most common (~43 %) while unavailable listings are rare.  Counts:
  
  |Availability (days)|Count|
  |---|---|
  |0|190|
  |1–100|2966|
  |101–200|3566|
  |201–300|5440|
  |301–365|9117|

* **`recency_bin`** – Discretises `days_since_last_review` into four categories: up to 30 days, 31–180 days, 181–365 days and more than 365 days.  Listings with a recent review (≤30 days) constitute about 16 % of the sample.  Counts:
  
  |Recency (days)|Count|
  |---|---|
  |<=30|3359|
  |31–180|10541|
  |181–365|2148|
  |>365|5231|

These bins allow models, especially linear ones, to learn non‑linear effects without complex transformations.  Tree‑based algorithms will also benefit from the categorical splits.

### Categoricals retained

The original categorical variables `neighbourhood_group`, `neighbourhood` and `room_type` were kept as category types.  These will be **one‑hot encoded** in the modeling phase.  We decided against reducing the number of neighbourhood categories at this stage because rare categories might still carry information.  In practice, high‑cardinality features can be encoded via frequency or target encoding; we will revisit this in feature selection (Chunk 9).

## Decision Rationale

* **Binning vs. continuous transformations** – Discretising variables such as review counts and host listing counts captures threshold effects that may not be linear.  We could model non‑linearity via polynomial terms or splines, but bins are simple, interpretable and less prone to overfitting.  We selected cut points based on domain knowledge (e.g., 0 reviews, 1–5 reviews) and distribution percentiles.
* **Log transforms** – Skewed numeric variables benefit from log1p transformations.  This technique compresses extreme values without discarding data and often enhances linear model performance.  Alternatives like Box–Cox or Yeo–Johnson were considered but require parameter estimation and can complicate interpretability.  Log1p is simple and effective.
* **Capping minimum nights** – We capped `minimum_nights` at 90 (99th percentile) instead of using the IQR rule (which erroneously flagged any value not equal to 30 as an outlier).  This prevents extremely long stays from dominating the log transformation.
* **Recency feature** – Binning `days_since_last_review` into broad categories strikes a balance between granularity and sample size within each bin.  Including the continuous log‑transformed version allows models to capture finer trends if needed.
* **Preparing for encoding** – High‑cardinality categorical variables will increase feature dimensions when one‑hot encoded.  Rather than prematurely reducing categories, we will evaluate their contribution during feature selection.  Rare levels may be grouped later if they prove uninformative.

## Requirements Recap

This chunk engineered new features from the cleaned dataset.  We added winsorised price variables, log‑transformed several skewed predictors, discretised continuous variables into meaningful bins and prepared categorical variables for encoding.  The resulting feature‑enhanced dataset has 26 columns and is stored as **{{file:file-NxEcsLUg7stUT8zJMAvQP4}}** for reproducibility.

## Mindmap & Where We Are

* **Outlier Analysis** – Completed (Chunk 7).
* **Feature Engineering** – Completed (this chunk).  New features capture host behaviour, review dynamics and price scaling.
* **Next steps** –  
  - **Feature Selection** (Chunk 9): evaluate feature importance using filter methods (correlation, mutual information) and model‑based approaches (e.g., random forest feature importances).  
  - **Modeling** (Chunks 10–11): build baseline and advanced regression models to predict price and log‑price.

## Compute Notes

* Feature engineering was executed on the full dataset (~21 k rows), yielding a 26‑column DataFrame (~5 MB).  Bin assignments and log transformations used vectorised Pandas operations, incurring negligible computational overhead.  The engineered dataset will be used in subsequent modeling steps.
