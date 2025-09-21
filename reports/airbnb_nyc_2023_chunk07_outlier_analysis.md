# CHUNK 7 – Outlier Analysis & Processing

## Objectives

Outliers can distort model training, inflate error metrics and obscure genuine relationships.  This chunk assesses extreme values in the cleaned dataset and formulates policies to handle them.  We examine numeric features using the inter‑quartile range (IQR) rule and propose capping strategies to mitigate undue influence while retaining as much data as possible.

## Outlier Identification

We applied the **1.5 × IQR rule** to each numeric feature in the processed dataset (21 279 listings).  For a variable with first quartile (Q1) and third quartile (Q3), values below `Q1 – 1.5 × IQR` or above `Q3 + 1.5 × IQR` are flagged as outliers.  The table summarises the number and percentage of outliers per feature:

|Variable|# Outliers|% Outliers|
|---|---|---|
|price|1690|7.94%|
|minimum_nights|5761|27.07%|
|number_of_reviews|2478|11.65%|
|reviews_per_month|2842|13.36%|
|calculated_host_listings_count|4012|18.85%|
|availability_365|0|0.00%|
|number_of_reviews_ltm|3099|14.56%|
|days_since_last_review|1906|8.96%|

Several features show high percentages of outliers.  However, context matters:

* **Price** – 7.9 % of listings have prices above \$507 (Q3 + 1.5 × IQR).  Many of these correspond to luxury listings or hotel rooms.  Removing them entirely would discard valuable information.  Instead, we consider capping extreme prices at a high percentile.
* **Minimum nights** – Q1 and Q3 both equal 30, making IQR = 0 and flagging any value other than 30 as an outlier.  This demonstrates the limitations of the IQR rule for discrete distributions.  Rather than removing 27 % of records, we will handle extreme minimum stays by capping them at a reasonable threshold (e.g., 90 nights, the 99th percentile).
* **Review counts and host listings** – Large numbers of reviews or host listings may legitimately reflect popular or professional hosts.  Logarithmic transformation (performed later) will reduce their skewness; we avoid dropping these cases.
* **Availability** – No availability outliers are detected.  Values are bounded between 0 and 365 by design.

### Price outlier handling

To reduce the influence of extreme prices without discarding data, we **winsorised** the top 5 % of prices.  Specifically, prices above the 95th percentile (≈\$615) were capped at this value.  Only 1 062 listings were affected.  Mean nightly price dropped from \$447 to \$202 after capping, while the median remained \$150.  The boxplot below contrasts the raw price distribution with the capped distribution; the capped version eliminates the long right tail but preserves the interquartile range.

![Boxplots comparing raw and capped price distributions]({{file:file-Cz2om3HYGger1fti98PDg3}})

We retained the capped price as a new variable (`price_capped`) and computed its logarithm (`log_price_capped`).  Models can compare performance using either the original or capped price; tree‑based models are robust to outliers but linear models benefit from capping.

### Minimum nights handling

The vast majority of listings require a 30‑night minimum stay.  Extreme values (up to 1 124 nights) likely correspond to long‑term rentals.  Instead of using the IQR rule, we propose capping `minimum_nights` at **90** (the 99th percentile).  This retains legitimate variation (1–90 nights) while preventing very long stays from skewing models.  A binary feature indicating whether a listing requires more than 30 nights may also prove predictive and will be engineered in a later chunk.

### Other numeric features

For `number_of_reviews`, `reviews_per_month`, `calculated_host_listings_count`, `number_of_reviews_ltm` and `days_since_last_review`, we do not remove outliers.  These variables capture genuine heterogeneity among listings.  We will apply **log1p transformations** in the feature engineering stage to reduce skewness and compress extreme values.  This approach preserves ordering information without truncation.

## Policy & Sensitivity

* **Capping vs. removal** – Removing price outliers would discard nearly 8 % of observations and bias the dataset towards mid‑range listings.  Capping (winsorisation) preserves sample size and reduces the influence of extreme values.  For minimum nights, the IQR rule flagged too many values; thus we chose a percentile‑based cap instead.  These policies balance robustness with data retention.
* **Sensitivity analysis** – After capping price, the mean dropped substantially while the median remained stable, indicating that outliers mainly affect the mean.  Modeling both capped and uncapped price will help assess sensitivity.  For minimum nights, capping at 90 will slightly reduce variance without altering the median.
* **Alternative methods** – Robust z‑scores (based on median and median absolute deviation) could identify outliers in skewed distributions; however, percentile capping is simpler and sufficient.  Transformations such as Box–Cox or Yeo–Johnson were considered but deferred because they can complicate interpretability.

## Requirements Recap

This chunk identified outliers using the IQR rule, highlighted variables where the rule is inappropriate, and proposed capping strategies.  We winsorised prices above the 95th percentile, capped extreme minimum nights at 90, and decided to log‑transform other skewed features rather than remove them.  These policies will improve model robustness without sacrificing sample size.

## Mindmap & Where We Are

* **Cleaning & Pre‑processing** – Completed (Chunk 6).
* **Outlier Analysis & Processing** – Completed (this chunk).  Decisions on price capping and handling of other variables have been recorded.
* **Next steps** –  
  - **Feature Engineering** (Chunk 8): create additional features (amenity counts, text lengths, ratios) and encode categorical variables.  
  - **Feature Selection** (Chunk 9): evaluate feature importance and reduce dimensionality.

## Compute Notes

* All outlier calculations were performed on the 21 279‑row cleaned dataset (approx. 4 MB).  Identifying IQR bounds and computing percentiles required negligible compute.  The capped price variable and updated dataset will be stored for modeling in subsequent chunks.
