# CHUNK 3 – Univariate Exploratory Data Analysis (EDA)

## Setup

In the previous chunk we audited the schema and confirmed that the **NYC Airbnb dataset** contains 36 403 listings and 18 fields (after selecting the appropriate snapshot).  To reduce memory pressure we loaded only the necessary columns using explicit `dtype` maps (downcasting integer counts to `int16/32` and real numbers to `float32`) and converted string‐like columns to Pandas `category` types.  The final in‑memory footprint is ~3.7 MB, well within our 12 GB budget.  Throughout this chunk a random seed (`np.random.seed(42)`) was used where sampling was required.

## Univariate Analysis of Numeric Variables

The target variable `price` and other numeric fields show heavy right‑skewness.  Logarithmic transformation is often used when a variable has a long right tail; the objective is to make the distribution more symmetric【559312119136520†L120-L124】.  The aim of any transformation is not to “fix” the data but to produce a reasonably symmetric distribution for downstream modelling【559312119136520†L169-L177】.  In this step we compute summary statistics, visualise distributions and record our observations.  Missing values were dropped for the summaries below.

### Summary statistics

|Variable|Count|Mean|Median|Std|Min|Q1|Q3|Max|Skew|Kurtosis|
|---|---|---|---|---|---|---|---|---|---|---|
|price|21279|447.87|150.0|3174.21|3.0|90.0|257.0|50052.0|14.2|204.9|
|minimum_nights|36403|28.62|30.0|29.29|1|30.0|30.0|1124|15.55|401.02|
|number_of_reviews|36403|26.85|3.0|68.39|0|0.0|22.0|3518|11.42|352.38|
|reviews_per_month|25093|0.82|0.25|1.88|0.01|0.08|0.92|123.87|20.02|924.28|
|calculated_host_listings_count|36403|63.41|2.0|202.49|1|1.0|9.0|1087|4.18|17.36|
|availability_365|36403|161.66|153.0|147.35|0|0.0|318.0|365|0.14|-1.65|
|number_of_reviews_ltm|36403|4.01|0.0|19.89|0|0.0|1.0|1771|36.22|2551.11|

* **Price** – 21 279 observations with non‑null prices.  The median price is **$150**, with an interquartile range (IQR) of $90–$257.  The long tail pushes the mean up to **$447**, and the maximum recorded price is **$50 052**, resulting in extreme skewness (14.2) and kurtosis (204.9).
* **Minimum nights** – Most listings require at least 30 nights, though a small number allow as few as one night.  A few extreme values (max = 1 124 nights) inflate the mean and skewness.
* **Review counts** – Both `number_of_reviews` and `reviews_per_month` are highly skewed; most listings have few reviews, but some exceed thousands of reviews.  The variable `availability_365` is comparatively symmetric (skew ≈ 0.14) and bounded between 0 and 365 days.

### Price distribution

The histogram below (left panel) shows the raw price distribution.  Nearly half of the listings cost below \$150 per night, while a small number of luxury properties command very high rates.  The long tail justifies testing a **logarithmic transformation**: the right panel plots `log1p(price)`, which compresses extreme values and yields a more symmetric shape.  This transformation improves interpretability and will be considered when building regression models.

![Histogram of price and log1p(price) distributions]({{file:file-BLVaz9sKq3R1FzYSw8sChC}})

### Other numeric variables

`minimum_nights` and `number_of_reviews` share similar heavy tails, with median values far below their means.  These variables may benefit from transformations (e.g., square root or log1p) or clipping when modelling.  `availability_365` shows that many listings are available either year‑round or rarely (0 days), indicating potential supply constraints.

## Univariate Analysis of Categorical Variables

### Room type distribution

Airbnb categorises accommodations into four room types.  The majority of NYC listings offer **entire homes or apartments** (≈53 %), while **private rooms** account for ~45 %.  Hotel and shared rooms are relatively rare.

![Room type distribution]({{file:file-552KXLEQhrEq7rVFACGxKc}})

### Neighbourhood groups

Listings cluster within the five boroughs.  Manhattan and Brooklyn together host over 80 % of listings; Queens provides 5 k listings, while the Bronx and Staten Island offer far fewer.  These counts provide context for subsequent spatial analyses.

![Neighbourhood group distribution]({{file:file-PrxVNz9vq7vcPECsLMmitu}})

### Top neighbourhoods

Among the 187 neighbourhoods, **Bedford‑Stuyvesant** (Brooklyn), **Williamsburg** (Brooklyn) and **Midtown** (Manhattan) have the highest listing counts (>2 000 each).  Harlem, Hell’s Kitchen and Bushwick follow closely.  These clusters may reflect tourist demand and host density.

## Decision Rationale

* **Histogram vs. boxplot** – Histograms were chosen for price and log‑price because they reveal the shape of the entire distribution, highlighting the extreme tail.  Boxplots are effective for identifying outliers but can compress information when extreme skewness is present.  An alternative would be violin plots or kernel density estimates; however, histograms with 50 bins strike a balance between detail and interpretability.  The x‑axis was truncated at \$1 000 to avoid the extreme tail dominating the display, ensuring clarity while still acknowledging the presence of outliers.
* **Log1p transformation** – We applied `log1p` (logarithm of 1 + price) to handle zero values and reduce skewness.  According to statistical best practices, log transformations are often used to reduce skewness and approximate symmetry【559312119136520†L120-L124】, improving modelling assumptions and enabling parametric techniques【559312119136520†L169-L177】.  Alternative transformations (square root, reciprocal) could have been considered; log1p is preferred for strictly positive financial data and produces interpretable percentage effects in regression.
* **Category ordering** – Bar plots were sorted by count to emphasise the most popular categories.  This aids the reader’s focus and simplifies comparisons.  Pie charts were avoided because they make it harder to compare similar categories.

## Requirements Recap

This chunk performed **univariate EDA** on the NYC Airbnb dataset (2023 snapshot).  We summarised numeric variables (price, review counts, minimum nights, availability) and categorical variables (room type, neighbourhood group).  We plotted raw and log‑transformed price distributions to address extreme skewness and provided descriptive statistics for each feature.  Decision rationales were supplied for choosing histograms, transformations and plot ordering.

## Mindmap & Where We Are

* **Project Charter** – defined objectives and CRISP‑DM roadmap (Chunk 0).
* **Business & Data Understanding** – contextualised the NYC Airbnb market and legal environment (Chunk 1).
* **Data Access & Schema Audit** – loaded the dataset with memory‑efficient dtypes, summarised the schema and missingness (Chunk 2).
* **Univariate EDA (this chunk)** – characterised the distribution of each variable, visualised price and categorical counts.
* **Upcoming** – next we will perform **bivariate and temporal EDA** (Chunk 4), exploring relationships between price and other features, including seasonality and host characteristics.

## Compute Notes

* **Memory efficiency** – Only the 18 available columns were loaded; numeric types were downcast and categorical variables encoded to save memory (~3.7 MB).  This allowed computations over the full dataset without sampling.
* **Plotting** – Visualisations were generated with Seaborn/Matplotlib.  To avoid memory spikes, each figure was saved and closed before generating the next.  Bins were limited to 50 and axes truncated where necessary to maintain readability.
* **Sampling** – Full data were used here because of the manageable size.  In later chunks (e.g., SHAP analyses), we will sample to respect the 12 GB limit.
