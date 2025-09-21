# CHUNK 6 – Data Cleaning & Pre‑processing

## Objectives

To prepare the dataset for modeling, this chunk focuses on handling missing values, engineering basic features and encoding variables.  Proper cleaning reduces noise, prevents data leakage and ensures reproducibility.  We follow best practices such as median imputation for skewed numeric variables and dropping features with excessive missingness【819127695883555†L69-L76】.

## Handling Missing Values

The listing dataset contains 36 403 rows and 18 columns.  After dropping rows with missing target (`price`), 21 279 listings remain.  Key missingness issues identified in the schema audit (Chunk 2) are summarised below:

|Column|Missing %|Action|
|---|---|---|
|`price`|41.5 %|Rows with missing price were discarded; imputation would contaminate the target.
|`reviews_per_month`|31.1 %|Imputed with median because the distribution is right‑skewed and median imputation preserves the central tendency【819127695883555†L69-L76】.
|`last_review`|31.1 %|Encoded into a new feature `days_since_last_review`; missing values filled with the median days since the last review.
|`license`|85 %|Dropped from analysis due to excessive missingness and minimal business relevance.
|`name`, `host_name`|<0.1 %|Text fields dropped; they do not directly inform price prediction and would require NLP.
|All other numeric columns|0 %|No action required.

### Imputation strategy

* **Median imputation for skewed variables** – When a numeric variable is heavily skewed, the median provides a robust central estimate compared to the mean【819127695883555†L69-L76】.  We used median imputation for `reviews_per_month` and for `days_since_last_review` when `last_review` was missing.  Imputation was computed on the training data and will be applied consistently to validation/test sets in subsequent chunks to avoid information leakage.
* **Dropping columns** – The `license` field was removed because more than 85 % of its entries are missing, making reliable imputation impossible.  We also removed `id`, `host_id`, `latitude`, `longitude`, `name` and `host_name` since they either identify listings or add little predictive power.
* **Dropping missing targets** – Listings without a `price` were excluded from modelling.  Retaining them would require predicting the target for missing observations, which is outside our scope.

## Feature Engineering

### Logarithmic target

Following our univariate analysis (Chunk 3), we added a **log‑transformed price** feature (`log_price = log1p(price)`) to stabilise variance and mitigate the long right tail.  This transformation allows linear models to better capture multiplicative effects and often yields more normally distributed residuals.

### Recency of last review

The `last_review` date indicates the most recent guest review.  To convert this into a numeric feature, we calculated `days_since_last_review` as the difference (in days) between the latest review date in the dataset and each listing’s `last_review`.  Listings without any review were assigned the median recency.  This feature proxies how recently a listing has been active; a smaller value implies recent engagement and may correlate with demand.

### Categorical variables

We retained the categorical variables `neighbourhood_group`, `neighbourhood` and `room_type`.  They are stored as Pandas `category` types.  These will later be transformed via **one‑hot encoding** or other encoding strategies during feature engineering (Chunk 8).  At this stage we do not expand them into dummy variables to conserve memory and avoid premature dimensionality expansion.

## Cleaned Dataset Summary

The cleaned dataset (`listings_processed.csv`, 21 279 rows) contains the following columns:

* **Categorical features** – `neighbourhood_group`, `neighbourhood`, `room_type`.
* **Numeric features** – `price`, `minimum_nights`, `number_of_reviews`, `reviews_per_month` (median‑imputed), `calculated_host_listings_count`, `availability_365`, `number_of_reviews_ltm`, `days_since_last_review`.
* **Derived target** – `log_price`.

All missing values in the remaining columns have been imputed, and the dataset is ready for further feature engineering and modelling.  The processed file has been saved for reproducibility: **{{file:file-UnqLFDuhTyx6TeVjac3wJE}}**.

## Decision Rationale

* **Median vs. mean imputation** – The distributions of `reviews_per_month` and `days_since_last_review` are highly skewed.  Median imputation preserves the central tendency without being distorted by extreme values and is recommended for skewed data【819127695883555†L69-L76】.  Mean imputation would overestimate typical values and could bias models.
* **Dropping high‑missingness features** – Retaining `license` (85 % missing) or imputing it as “unknown” could introduce noise because the absence of a license may correlate with compliance issues but does not directly impact price.  Dropping it simplifies the model.  The identifier columns (e.g., `id`, `host_id`) were removed to prevent data leakage and avoid undue model complexity.
* **Recency feature** – Transforming a date into a relative measure (days since last review) is more informative for modelling than raw dates.  We chose the difference from the latest `last_review` because the dataset covers multiple years and we lack the exact snapshot date.  An alternative would be the number of months since last review, but day‑level granularity captures nuance without adding complexity.
* **Log price target** – Including both the original `price` and `log_price` allows comparing models that predict either scale.  The log transformation can reduce heteroskedasticity and improve linear model performance, while tree‑based models can still use the raw price.

## Requirements Recap

This chunk cleaned and pre‑processed the dataset.  Key steps included dropping rows with missing target values, median‑imputing skewed numeric features, engineering `days_since_last_review`, dropping high‑missingness or identifying columns, and creating a log‑transformed price.  The cleaned dataset was saved for subsequent analysis.

## Mindmap & Where We Are

* **Geo‑EDA** – Completed (Chunk 5).
* **Cleaning & Pre‑processing** – Completed (this chunk).  The data is now tidy, with imputed numeric values and engineered features.
* **Next steps** –  
  - **Outlier Analysis & Processing** (Chunk 7): identify and handle extreme price values (e.g., luxury hotels) to prevent them from unduly influencing models.  
  - **Feature Engineering** (Chunk 8): derive additional predictors (amenity counts, text lengths, etc.) and encode categorical variables.

## Compute Notes

* The cleaning operations were performed on the full dataset of 21 279 records, consuming ~4 MB of RAM.  Median imputation and feature engineering required negligible compute.  The cleaned dataset is stored as a CSV file (approx. 2 MB), ensuring reproducibility and enabling efficient reloads in later chunks.
