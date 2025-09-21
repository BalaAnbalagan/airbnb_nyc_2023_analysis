# Unlocking New York City Airbnb Pricing: A Data‑Science Journey

**How host scale, room type and neighbourhood shape nightly rates – and what it means for guests, hosts and regulators**

Few cities have grappled with the impact of short‑term rentals as much as New York.  In 2023 the city introduced Local Law 18, requiring hosts to register and abide by strict occupancy rules.  At the same time, travellers returned post‑pandemic, and property owners sought to maximise revenue.  To understand the dynamics behind nightly prices, we analysed a public snapshot of **21 000 NYC Airbnb listings** scraped in March 2023【34995615265008†L44-L46】.  Our goal: discover what drives prices, build predictive models and uncover hidden segments.  Here’s what we found.

## 1. From Raw Data to Clean Features

We began by loading the `listings.csv` file from Inside Airbnb and auditing its structure.  Price values were strings with dollar signs and commas; several fields (like `license` and `last_review`) were sparsely populated.  A missingness heatmap (below) highlighted columns needing attention – notably, ~42 % of `price` values were missing and many reviews were absent.  We removed listings with missing prices or converted text fields to categories and numbers.  After cleaning we retained 21 279 listings and engineered new features: capped prices (to limit the impact of outliers), log‑transformed prices and counts (to reduce skewness), and bins for host size, review counts and availability.  Median imputation was preferred for skewed numeric variables【819127695883555†L69-L76】.

![Missing data heatmap]({{file:file-L3RFqnYnnDyB5YZbbuUzwy}})

### Who and where are these listings?

Manhattan and Brooklyn dominate the supply, while private rooms account for nearly half of listings.  The charts below show the distribution of room types and boroughs, underscoring how concentrated the market is in a few central areas.

![Room type distribution]({{file:file-552KXLEQhrEq7rVFACGxKc}})

![Neighbourhood group distribution]({{file:file-PrxVNz9vq7vcPECsLMmitu}})

## 2. Exploring Pricing Patterns

Nightly rates ranged from \$0 (free or test listings) to more than \$500.  The raw price distribution was extremely skewed, so we also looked at the log of price (using `log1p`).  The log transformation produced a more symmetric distribution【559312119136520†L120-L124】【559312119136520†L169-L177】, which later improved model performance.

![Raw vs. log price distribution]({{file:file-BLVaz9sKq3R1FzYSw8sChC}})

Correlations revealed that higher prices were associated with hosts owning more listings, longer minimum stays and greater annual availability.  The heatmap below displays pairwise correlations among key numeric features.  Notice that host listing count and minimum nights show moderate correlation with price, while review‑based features correlate only weakly.

![Correlation matrix]({{file:file-ACJWsEhVWNUc6GsD8tVLza}})

Bivariate plots confirmed these relationships.  For example, prices climb as the host’s portfolio size increases and decline as minimum nights rise (particularly beyond 30 days).  There was no strong linear relationship between price and review counts or recency, suggesting that other factors dominate pricing decisions.

![Price vs. host and stay requirements]({{file:file-TtBRx7Z83QKGXAh3JPy1BS}})

A simple temporal analysis using the `last_review` date showed no obvious price trend over time, but it did reveal a decline in review frequency after early 2020, reflecting pandemic disruptions.

## 3. Mapping the Market

Using anonymised coordinates (Airbnb randomly shifts each location by up to 150 m【788864212863535†L40-L44】), we created hexbin maps to visualise where listings concentrate and how prices vary across the city.  The density map highlights the **Manhattan core and northern Brooklyn** as hotspots, while the average price map shows that **central Manhattan commands the highest rates**, with pockets of high pricing in tourist‑heavy areas like Williamsburg and Chelsea.

![Listing density by hexagon]({{file:file-DVxGejgUrLMpHDKK654qDt}})

![Average price by hexagon]({{file:file-WTCbFPPtZ7NsrqQrVjAcxN}})

## 4. Cleaning & Processing: Handling Outliers and Missingness

Outliers can distort model estimates and mislead conclusions.  We applied the IQR rule to flag unusually high prices and capped the nightly rate at \$500 (which affected ~1 % of listings).  The boxplot below compares the original and capped price distributions.  We also converted skewed variables to logs and created indicator variables for extreme minimum nights and host sizes.

![Outlier treatment comparison]({{file:file-Cz2om3HYGger1fti98PDg3}})

## 5. Building Predictive Models

### Baselines and linear models

Before tackling sophisticated algorithms, we measured two naive baselines: predicting the mean and median price.  Both baselines performed poorly (R²≈0), highlighting the need for modeling.  Ordinary least squares, Ridge and Lasso regressions raised R² to **~0.57** when predicting log prices and ~0.51 on the raw scale.  Residual plots hinted at remaining non‑linear structure.

![Predicted vs. actual log prices]({{file:file-Aa1hJXLqm2K9KUX6at7uh3}})

![Residuals vs. predicted log prices]({{file:file-TQrq9wM1woXng4x58fStcR}})

### Tree ensembles

To capture complex interactions, we fitted a **Random Forest** and a **Gradient Boosting** regressor on the log‑scaled target.  These models increased R² to **≈0.66** and reduced RMSE by about 10 %.  When exponentiating predictions back to dollars, the Random Forest achieved an RMSE of \$99.75 and R²≈0.58, outperforming the linear model trained on raw prices.  Feature importances from the Random Forest echoed earlier findings: room type, host portfolio size, minimum nights and availability dominated.  The bar chart below shows the top predictors.

![Random forest feature importances]({{file:file-9FACoNzD3YGf2JqapT8uDp}})

## 6. Segments Beneath the Surface: Clustering Results

Unsupervised learning revealed nuanced patterns beyond price modelling.  A four‑cluster **K‑Means** solution on numeric features uncovered:

1. **Luxury Manhattan** – high prices (\$500+), entire homes, located in Manhattan and central Brooklyn.
2. **Active mid‑range** – moderate prices (\$139), frequent reviews and mixed room types.
3. **Typical** – majority of supply with average prices and typical host activity.
4. **Dormant** – small segment with long gaps since last review, suggesting inactive or long‑term rentals.

Separately, **DBSCAN** on standardized lat–lon coordinates identified nine geographic clusters.  The largest cluster covers the Manhattan/Brooklyn core, while smaller clusters correspond to Staten Island, Far Rockaway and Coney Island.  These spatial clusters highlight how Airbnb activity concentrates in the city centre but also flourishes in niche beach communities and suburban neighbourhoods.

![DBSCAN geo clusters]({{file:file-MUweFwHGCyJf9QhGn54WbL}})

## 7. Takeaways for Hosts and Policy Makers

Our analysis suggests several actionable insights:

* **Host scale and professionalism matter**: Expanding from a single listing to a portfolio can significantly increase revenue.  Professional hosts should ensure compliance with registration requirements and invest in hospitality quality to justify higher rates.
* **Room type is a key lever**: Private rooms attract budget‑conscious travellers and compete on price.  Entire homes draw higher‑paying guests seeking privacy and space.
* **Optimise minimum nights and availability**: Long minimum stays deter bookings and lower nightly rates.  Year‑round availability signals professionalism and supports premium pricing.
* **Location still rules**: Manhattan and tourist‑friendly Brooklyn neighbourhoods command the highest prices.  Peripheral areas like the Rockaways and Staten Island remain niche markets with distinct dynamics.
* **Monitor review activity**: Stale listings with no recent reviews risk being deprioritised.  Encourage guests to leave feedback and update listings regularly.

For regulators and platform managers, data‑driven segmentation can guide enforcement and policy.  Use host portfolio size and availability to prioritise inspections, tailor regulations to neighbourhood clusters, and provide transparency through public dashboards.

## Conclusion

Behind the nightly rate on an Airbnb listing lies a complex interplay of factors—from a host’s scale and listing type to the neighbourhood’s allure and the cadence of guest reviews.  By applying the CRISP‑DM framework to a real‑world dataset, we gained not just predictive accuracy but also human insights into the marketplace.  As New York continues to balance tourism, housing and regulation, data science offers a compass for hosts seeking success and a lens for regulators striving for fairness.
