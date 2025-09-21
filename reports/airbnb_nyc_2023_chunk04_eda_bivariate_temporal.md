# CHUNK 4 – Bivariate & Temporal EDA

## Goals

This chunk extends the exploratory analysis by examining relationships between the nightly price and other features.  We explore pairwise correlations, visualise how price varies across room types and boroughs, and investigate whether review activity, minimum‑stay requirements or availability influence price.  A simple temporal view using the `last_review` date helps assess market evolution over the last decade.

## Bivariate Relationships

### Correlation matrix

The Pearson correlation matrix of numeric variables is visualised below.  Price shows very weak linear relationships with other numeric attributes (coefficients between −0.08 and 0.03), suggesting non‑linear or categorical drivers are more influential.  In contrast, `number_of_reviews` and `reviews_per_month` are strongly correlated (0.90) because `reviews_per_month` derives from total reviews over time.  `number_of_reviews_ltm` (reviews in the last 12 months) also correlates moderately with total reviews (0.65), highlighting collinearity among review metrics.

![Correlation matrix heatmap]({{file:file-ACJWsEhVWNUc6GsD8tVLza}})

### Price by room type and borough

Median nightly prices vary markedly by room category.  **Hotel rooms** exhibit extremely high medians (~\$40 k) because the dataset includes a few luxury hotels; these are outliers and will be treated cautiously.  Excluding hotel rooms, **entire homes/apartments** command the highest median price (~\$203), followed by **private rooms** (~\$88) and **shared rooms** (~\$65).  By borough, **Manhattan** listings are most expensive (median \$211), followed by **Brooklyn** (\$129), **Queens** and **Staten Island** (~\$100) and the **Bronx** (~\$94).  These patterns align with expected differences in demand and neighbourhood prestige.

### Price vs. review activity, stay length and availability

To visualise continuous relationships we sampled 2 000 listings (seed = 42) and plotted log‑transformed price against log‑transformed `number_of_reviews`, log‑transformed `minimum_nights` and raw `availability_365`.

![Scatter plots for price vs. review count, minimum nights and availability]({{file:file-TtBRx7Z83QKGXAh3JPy1BS}})

* **Number of reviews** – There is no clear relationship; listings with few reviews can be either cheap or expensive.  A slight downward trend suggests that highly reviewed properties tend to have lower prices, possibly because budget listings attract more bookings.  The negative Pearson correlation (−0.03) supports this interpretation.
* **Minimum nights** – Higher minimum stays are associated with lower nightly prices (correlation −0.08).  Long‑stay listings often target residents rather than tourists, so hosts lower nightly rates to secure extended bookings.
* **Availability_365** – Availability shows almost no linear correlation with price (0.03).  Binning availability into categories reveals subtle effects: listings with zero availability (booked out or blocked) have the highest median price (≈\$157), whereas those available for 1–100 days tend to be slightly cheaper (~\$138).  However, differences are modest, indicating that availability alone does not strongly drive price.

### Temporal patterns

Although the dataset contains only a snapshot of prices and no booking dates, the `last_review` field records when guests last reviewed each listing.  Grouping by review year shows how median price has shifted over time.  Prices were higher in 2011–2013 (median ≈\$213) but have stabilised around \$150 since 2015.  The slight dip in 2024 may reflect regulatory changes such as **New York City’s Local Law 18**, which tightened short‑term rental regulations in September 2023.  The plotted trend serves as a qualitative indicator rather than evidence of causation.

![Median price by review year]({{file:file-LvieKUJouXTQaSdUxpsZFX}})

## Decision Rationale

* **Correlation analysis** – We computed Pearson correlations to quantify linear relationships between variables.  Alternative measures (Spearman correlation or mutual information) could capture non‑linear associations, but Pearson suffices for an initial screening.  The heatmap uses diverging colours centered at zero to emphasise both positive and negative associations.
* **Median vs. mean** – When comparing price across categories, we used the median rather than the mean because of extreme outliers (e.g., luxury hotels).  The median is robust to skewness and better reflects typical prices.  Means would be inflated and could mislead stakeholders.  For review counts and availability bins we similarly relied on medians to avoid dominance by a few high‑priced listings.
* **Sampling for scatter plots** – Plotting all 21 279 price observations would crowd the figure and slow rendering.  A simple random sample of 2 000 points preserves patterns while respecting compute constraints.  We used `log1p` transformations on axes with highly skewed variables to compress extreme values and reveal underlying trends.
* **Temporal analysis** – `last_review` provides the only time signal in the data.  We aggregated by year instead of month to reduce noise and protect privacy.  Without per‑booking price history we cannot infer true temporal price changes; hence, we present the trend cautiously and avoid over‑interpretation.

## Requirements Recap

In this chunk we explored **bivariate and temporal relationships**.  We visualised the correlation matrix of numeric features and highlighted weak linear associations between price and other variables.  We compared median prices across room types and boroughs, examined scatter plots of price against review counts, minimum stays and availability, and analysed simple temporal trends using the year of the last review.  Decision rationales explained our use of medians, log transformations, sampling and the limitations of the temporal analysis.

## Mindmap & Where We Are

* **Univariate EDA** – Completed (Chunk 3).  We now understand the individual distributions of all key variables.
* **Bivariate & Temporal EDA** – Completed (this chunk).  We have identified important relationships (e.g., room type and borough matter; review counts and minimum nights have weak negative associations with price).
* **Next steps** –  
  - **Geo‑EDA** (Chunk 5): examine spatial patterns of price across neighbourhoods, potentially using hex bins or choropleths, and compute aggregated metrics.  
  - **Data cleaning & preprocessing** (Chunk 6): impute missing values, encode categorical variables, and scale numeric features in preparation for modeling.

## Compute Notes

* **Memory management** – The correlation matrix and summary statistics were computed on the full dataset (3.7 MB).  For scatter plots we sampled 2 000 observations to limit plotting overhead.  All figures were generated sequentially and saved at 150 dpi to balance clarity and file size.
* **Temporal aggregation** – Yearly aggregation reduced the number of groups from 138 months to 15 years, allowing a simple trend line without overfitting.  Missing `last_review` entries (31 % of listings) were excluded from temporal analysis.
