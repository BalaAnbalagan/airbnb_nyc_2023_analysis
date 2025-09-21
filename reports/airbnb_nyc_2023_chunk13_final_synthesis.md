# CHUNK 13 – Final Synthesis & Recommendations

## Executive Summary

This project examined the Inside Airbnb 2023 snapshot for New York City to understand factors that influence nightly prices and to derive actionable insights for hosts and regulators.  Using **CRISP‑DM** as a guide, we conducted rigorous data cleaning, explored univariate and bivariate patterns, modeled prices with linear and tree‑based methods, and applied clustering to discover segments.  The analysis shows that price varies widely across listings (median \$150, range \$0–\$500+), is highly skewed and concentrated in Manhattan, and depends on **host scale**, **room type**, **minimum nights**, **availability**, **location** and **review activity**.

### Key Insights

* **Host portfolio size drives price**: Listings owned by hosts with many properties command higher prices; `log_host_listings_count` and `calculated_host_listings_count` had the highest mutual information and feature importance across models.  This suggests professional hosts can charge more due to experience, amenities or marketing.
* **Room type matters**: Private rooms are significantly cheaper than entire homes; the `room_type_Private room` dummy was the single most important feature in tree models, accounting for ~30 % of variance explained.
* **Minimum nights and availability**: Higher minimum night requirements correlate negatively with price (especially when expressed on a log scale), whereas greater annual availability correlates positively.  Listings open year‑round command higher nightly rates but may also reflect full‑time rentals.
* **Location effects**: Manhattan listings are consistently pricier.  Geospatial clustering identified a core Manhattan/Brooklyn cluster and several peripheral clusters (Staten Island, Far Rockaway, Coney Island).  The luxury segment identified by K‑Means aligns with the geographic core.
* **Review recency and intensity**: Listings with more recent and frequent reviews tend to be cheaper, perhaps reflecting a focus on occupancy over price.  Long periods since the last review flagged a small “dormant” cluster, suggesting possible non‑compliance or long‑term rentals.
* **Model performance**: Linear regression (with Ridge/Lasso) explained ~57 % of variance on the log scale and beat naive baselines, but tree ensembles improved performance further (R²≈0.66) and reduced RMSE by ~10 %.  Transforming prices to the log scale improved model fit due to reduced skewness【559312119136520†L120-L124】【559312119136520†L169-L177】.
* **Segments**: K‑Means uncovered four behavioural clusters—luxury Manhattan, active mid‑range, typical and dormant—while DBSCAN separated Manhattan/Brooklyn from peripheral neighbourhoods.  These segments can guide pricing and marketing strategies.

## Business Recommendations

### For Hosts

1. **Benchmark and adjust prices**: Compare your listing’s features to those identified as price drivers.  If you offer an entire home in Manhattan with high availability, you may fall into the luxury segment and can justify premium pricing.  Conversely, private rooms with frequent reviews should adopt dynamic pricing to balance occupancy and revenue.
2. **Optimize minimum nights**: Extremely long minimum stays lower your nightly rate.  Consider reducing minimum nights during high‑demand periods to attract short‑stay travellers.  Capping minimum nights at 30 or fewer aligns with typical traveller preferences.
3. **Expand thoughtfully**: Growing your host portfolio can raise your earning potential, but only if you maintain quality.  Large hosts are rewarded with higher prices, but they also face increased regulatory scrutiny.  Ensure compliance with Local Law 18 (host registration and safety requirements)【223133324145814†L61-L70】.
4. **Maintain listing activity**: Regular reviews signal active hosting and improve search rankings.  Listings with long gaps since the last review form a dormant cluster and may be deprioritized in search results.  Encourage guests to leave feedback and update your listing regularly.
5. **Enhance amenities and photos**: Our models did not include textual amenities due to compute constraints, but prior research indicates that amenities and quality photos impact price.  Investing in these areas can differentiate you within the typical cluster.

### For City Regulators & Platform Managers

1. **Targeted enforcement**: High‑priced, high‑availability listings concentrated in Manhattan and central Brooklyn likely belong to professional hosts.  Ensure these operators comply with registration and tax regulations.  The dormant cluster may hide unregistered long‑term rentals—spot checks could improve compliance.
2. **Neighbourhood‑based policies**: DBSCAN clusters show distinct peripheral markets (e.g., Far Rockaway, Staten Island).  Tailor regulations and support programs to the specific needs of these neighbourhoods, which may differ from Manhattan’s dense rental landscape.
3. **Monitor host portfolios**: Host portfolio size strongly influences price and may correlate with illegal hotel operations.  Use data to flag hosts with many listings for audits or targeted outreach.
4. **Encourage transparency**: Provide public dashboards summarizing listing distributions, prices and host activity by borough.  This can foster community trust and help regulators track progress towards housing goals.

## Limitations

* **Snapshot data**: The dataset represents a single scrape (March 6, 2023)【34995615265008†L44-L46】; prices and availability vary seasonally and may have changed following the implementation of Local Law 18 in September 2023.  Our models therefore capture average behaviour at the time of scraping and may not reflect post‑law dynamics.
* **Missing and anonymized data**: Listings with missing price or review data were removed, potentially biasing results.  Airbnb anonymizes coordinates within 150 m【788864212863535†L40-L44】; geospatial clusters are approximate and cannot reveal exact addresses.
* **Limited feature set**: We focused on numeric and categorical variables; textual descriptions, amenities and host ratings were omitted due to compute constraints.  Including natural‑language features could improve predictive power and interpretability.
* **Simplified modelling**: Hyperparameter tuning was limited to a few values for time reasons.  Ensemble methods such as XGBoost or LightGBM could yield better performance.  SHAP value analysis was not performed due to package availability; feature contributions were assessed via Gini importances instead.
* **Clustering subjectivity**: Selecting the number of K‑Means clusters and DBSCAN parameters involves trade‑offs.  Different choices could reveal alternate segmentations.  Our selection prioritized interpretability and reasonable silhouette scores.

## Reproducibility & Environment

The analysis was conducted in a Python 3.11 environment with the following key packages: pandas 1.5.3, numpy 1.24.0 and scikit‑learn 1.1.3.  Random seeds were set to 42 where applicable.  To reproduce the results:

1. Obtain the Inside Airbnb `listings.csv` file for New York City (March 2023 snapshot).
2. Follow the code provided in the per‑chunk notebooks or scripts to clean, engineer features, and model the data.  Ensure that numerical fields are parsed correctly (remove dollar signs and commas from price) and that categorical variables are encoded consistently.
3. Use `StandardScaler` for numeric features and `OneHotEncoder` for categorical features within a scikit‑learn `Pipeline` to prevent data leakage.
4. Fit baseline, linear (OLS/Ridge/Lasso) and tree ensemble models (Random Forest and Gradient Boosting) on an 80/20 train/test split.  Evaluate using RMSE, MAE and R².
5. Apply K‑Means clustering (k=4) on standardized numeric attributes and DBSCAN (eps=0.2, min_samples=20) on standardized lat–lon coordinates to obtain behavioural and spatial clusters.

## Conclusion

This project demonstrates how a systematic, CRISP‑DM‑aligned approach can yield rich insights from publicly available Airbnb data.  By combining descriptive analytics, predictive modelling and unsupervised learning, we identified factors driving nightly prices, quantified model performance and revealed meaningful host and geographic segments.  These findings support data‑driven decisions for both hosts aiming to optimise revenues and regulators seeking to balance tourism with housing availability.  Future work could incorporate text mining of reviews and amenities, dynamic pricing models and continuous monitoring as regulatory landscapes evolve.
