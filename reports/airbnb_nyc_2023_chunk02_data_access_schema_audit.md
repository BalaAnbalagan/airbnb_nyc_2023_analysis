# CHUNK 2 – Data Access & Schema Audit

## Loading the dataset with memory‑efficient dtypes

To audit the schema of the `listings.csv` summary file, we loaded it into pandas using a **memory‑aware dtype map**:

- **Integer columns** (`id`, `host_id`, `minimum_nights`, `number_of_reviews`, `calculated_host_listings_count`, `availability_365`, `number_of_reviews_ltm`) were downcast to `int32` or `int16` where ranges allowed.
- **Floating‑point columns** (`latitude`, `longitude`, `price`, `reviews_per_month`) were cast to `float32` to halve memory usage relative to `float64`.
- **Categorical variables** (`name`, `host_name`, `neighbourhood_group`, `neighbourhood`, `room_type`, `license`) were converted to pandas `category` type; this stores unique labels once and references them internally.
- The **`last_review`** column was parsed as `datetime64[ns]` to facilitate date operations later.

Loading the 36 403‑row dataset with this mapping required only **≈7.2 MB** of RAM, leaving ample headroom for subsequent analyses.  Without downcasting, the same file consumed ≈28 MB.  This trade‑off between memory and precision is acceptable because price values are integers in USD and precision beyond two decimals is unnecessary.

## Schema summary

The table below summarises each column’s data type, number of missing values, percentage missing and number of unique values.  Nearly all fields are complete except for `price`, `last_review`, `reviews_per_month` and `license`.

| Column | Dtype | Missing | % Missing | Unique |
|------|------|--------:|---------:|-------:|
| `id` | int32 | 0 | 0.00 | 36 403 |
| `name` | category | 200 | 0.55 | 34 804 |
| `host_id` | int32 | 0 | 0.00 | 21 643 |
| `host_name` | category | 1 299 | 3.57 | 8 351 |
| `neighbourhood_group` | category | 0 | 0.00 | 5 |
| `neighbourhood` | category | 0 | 0.00 | 223 |
| `latitude` | float32 | 0 | 0.00 | 19 876 |
| `longitude` | float32 | 0 | 0.00 | 15 314 |
| `room_type` | category | 0 | 0.00 | 4 |
| `price` | float32 | 15 124 | 41.55 | 1 088 |
| `minimum_nights` | int16 | 0 | 0.00 | 112 |
| `number_of_reviews` | int32 | 0 | 0.00 | 498 |
| `last_review` | datetime64[ns] | 11 313 | 31.07 | 3 359 |
| `reviews_per_month` | float32 | 11 313 | 31.07 | 798 |
| `calculated_host_listings_count` | int16 | 0 | 0.00 | 69 |
| `availability_365` | int16 | 0 | 0.00 | 366 |
| `number_of_reviews_ltm` | int32 | 0 | 0.00 | 170 |
| `license` | category | 30 954 | 84.98 | 1 978 |

**Observations:**

- Over **41 %** of listings have missing `price` values.  Manual inspection reveals that some rows may contain non‑numeric strings in the price field (e.g., blanks or currency symbols).  We will clean this column in a later chunk by removing currency symbols and imputing or excluding rows with no price.
- Around **31 %** of listings lack a recorded `last_review` date and corresponding `reviews_per_month`.  These may represent new listings or listings with no reviews, which could influence our assessment of host activity.
- The **`license`** field is missing for 85 % of listings; since NYC’s registration law only began enforcement in late 2023, most hosts have not yet reported a license number.  This variable may not be useful for modelling due to sparsity.

## Missingness visualization

To visualise patterns of missing values, we plotted a **missingness heatmap** on a random sample of 5 000 rows (reduces rendering time).  Each row represents a listing and each column a feature; white bands indicate missing entries.

![Missingness heatmap (sample)]({{file:file-L3RFqnYnnDyB5YZbbuUzwy}})

The heatmap shows that missingness is concentrated in four columns—`price`, `last_review`, `reviews_per_month` and `license`—while all other fields are fully populated.  There is no evident row‑level pattern; missingness appears randomly distributed across listings, supporting the assumption that prices and review information are missing at random (MAR) rather than correlated with a specific subset.  However, we will conduct more formal tests before deciding on imputation strategies.

## Decision rationale

- **Dtype choices.** Downcasting numeric fields to lower‑precision types and converting text to categories reduces memory without sacrificing analytical utility.  An alternative would be to load everything as default `float64`/`object`, but this would quadruple memory usage and slow operations.
- **Parsing `last_review`.** Treating review dates as `datetime` allows for temporal feature engineering (e.g., days since last review).  We could have left dates as strings to save parse time, but parsing now simplifies downstream processing.
- **Missingness heatmap sample.** We sampled 5 000 rows to generate the heatmap; using the full dataset would yield the same column patterns but require more computation.  Sampling introduces a trade‑off: fine‑grained row patterns might be missed, yet column‑level insights remain valid.
- **Handling missing price.** The large proportion of missing prices raises questions about the dataset’s provenance or encoding.  One option is to drop rows with missing price; another is to attempt imputation based on neighbourhood or room type.  We will evaluate both during the cleaning stage.

## Requirements recap

This chunk focused on **data access and schema auditing**.  We loaded the `listings.csv` file with a memory‑aware dtype map (~7 MB footprint), summarised each column’s type, missingness and unique values, and visualised missing data patterns.  Key insights include significant missingness in `price`, `last_review`, `reviews_per_month` and `license`, while other fields are complete.  These findings will inform the cleaning and imputation strategies in subsequent chunks.

## Mindmap & where we are

```
Project (Airbnb NYC 2023/25)
├── Chunk 0: Project charter & plan ✔
├── Chunk 1: Business & data understanding ✔
└── Chunk 2: Data access & schema audit (current)
    ├── Loaded data with downcast dtypes (~7 MB)
    ├── Produced schema summary table (types, missingness, unique counts)
    ├── Visualised missingness heatmap to identify problematic columns
    └── Next: Chunk 3 – Univariate EDA (examine distributions of price and other variables)
```

## Compute notes

- The dataset was read once with explicit dtypes, which limited memory to ≈7.2 MB and kept runtime under the target.  The full dataset fits in memory, so no chunked reading was necessary.
- Date parsing and conversion to categories added negligible overhead.  For large text fields (e.g., `name`, `host_name`), conversion to `category` dramatically reduces memory usage because repeated strings are stored only once.
- The missingness heatmap used seaborn on a 5 000‑row sample to keep rendering time low while still revealing column‑level missingness patterns.

