# CHUNK 12 – Clustering Analysis

## Objective

Beyond regression, clustering helps uncover distinct groups of Airbnb listings that share similar characteristics.  Identifying such segments can inform targeted marketing, pricing strategies and operational decisions.  This chunk explores **K‑Means** clustering on numeric listing attributes and **DBSCAN** clustering on geographic coordinates.

## K‑Means Clustering of Listing Attributes

### Methodology

* **Feature selection:**  We selected numeric attributes that capture price and operational characteristics: `price_capped`, `minimum_nights`, `calculated_host_listings_count`, `availability_365`, `reviews_per_month` and `days_since_last_review`.  These variables summarise nightly rate, stay requirements, host portfolio size, availability, demand intensity and recency of guest interaction.
* **Standardization:**  Because K‑Means relies on Euclidean distances, features were standardized (zero mean, unit variance) using `StandardScaler`.
* **Determining k:**  A sample of 5 000 listings was used to evaluate silhouettes for k=2…6.  Silhouette scores peaked at k=2 (≈0.53) but decreased modestly for k=3–5 (~0.27–0.31).  We chose **k=4** as a compromise between interpretability and cluster separation: more than two clusters provide richer segmentation while retaining reasonable cohesion.
* **Model:**  K‑Means was fitted with 4 clusters (`n_init=20`, `random_state=42`) on the full 21 279‑row dataset.  Cluster labels were appended to the listings for analysis.

### Findings

| Cluster | Share of listings | Median Price | Key characteristics |
|---|---|---|---|
| **Cluster 2 – “Luxury Manhattan”** | 4.6 % | **$509** | Comprised entirely of entire homes/apartments in Manhattan and central Brooklyn.  These listings have high prices, short minimum nights, high availability and moderate days since last review.  They represent high‑end, professional hosts. |
| **Cluster 1 – “Active, mid‑range”** | 38 % | $139 | Mixed room types (≈50 % private rooms).  These listings have relatively low minimum nights, frequent reviews (median 143 days since last review) and moderate host portfolio size.  They tend to be budget‑friendly yet active. |
| **Cluster 3 – “Typical”** | 52 % | $149 | Majority of listings with standard pricing and availability.  Host portfolios and review recency are middle‑of‑the‑road.  Includes both entire homes and private rooms across all boroughs. |
| **Cluster 0 – “Dormant/rarely reviewed”** | 5 % | $150 | Listings with extremely long times since last review (median ~5.4 years).  Could represent long‑term rentals or inactive hosts.  Mix of room types with moderate pricing and availability. |

Figure 4 presents cluster composition by borough and room type.  Cluster 2 is dominated by Manhattan, whereas cluster 1 and 3 include significant shares of Brooklyn and Queens.  Cluster 0 contains a disproportionate number of Bronx and Staten Island listings, reflecting lower demand.

### Business Interpretation

* **Luxury cluster:**  High‑end Manhattan listings could justify premium pricing strategies, enhanced concierge services or partnerships targeting affluent travellers.  Professional hosts may leverage economies of scale and invest in high‑quality amenities.
* **Active mid‑range cluster:**  These hosts attract frequent bookings and reviews.  Pricing strategies might focus on dynamic pricing based on seasonality and local events to maintain occupancy without sacrificing revenue.
* **Typical cluster:**  Represents the bulk of supply.  Standard pricing models apply, but hosts may benefit from differentiating through amenities or improved listing quality.
* **Dormant cluster:**  The long gap since last review suggests potential non‑compliance or listings used for other purposes (e.g., long‑term rentals).  Outreach could encourage these hosts to update their listings or ensure compliance with local regulations.

## DBSCAN Clustering of Geographic Coordinates

### Methodology

* **Data:**  Latitude and longitude from the original `listings.csv` were used.  Airbnb anonymizes coordinates by shifting them up to 150 m【788864212863535†L40-L44】, but spatial clustering still reveals approximate neighbourhood groupings.
* **Scaling:**  Coordinates were standardized to equalize the influence of latitude and longitude.
* **DBSCAN parameters:**  We experimented with `eps` values from 0.05 to 0.4 (in standardized units) and selected `eps=0.2`, `min_samples=20`.  This configuration produced **nine clusters** and a small noise set (~199 listings).  Smaller `eps` values yielded many tiny clusters with large noise; larger values collapsed distinct neighbourhoods into a single cluster.
* **Interpretation:**  Cluster centroids (median coordinates) indicate the approximate location of each group.  The largest cluster covers the Manhattan/Brooklyn core (latitude ~40.73, longitude ~−73.95).  Smaller clusters correspond to peripheral areas: Far Rockaway/Queens (cluster 1), Staten Island (clusters 2, 4, 7, 8) and Coney Island (clusters 3, 5, 6).  Noise points are scattered around the city.

![DBSCAN Geo Clusters]({{file:file-MUweFwHGCyJf9QhGn54WbL}})

*Figure 4 – DBSCAN clustering of 5 000 randomly sampled listings in the latitude–longitude plane.  Colours represent cluster IDs; noise points are labelled “−1”.  The largest cluster (0) covers the Manhattan/Brooklyn core, while smaller clusters delineate peripheral neighbourhoods such as Staten Island and the Rockaways.*

### Business Interpretation

* **Core vs. peripheral markets:**  The geographic clusters highlight how listings concentrate in Manhattan and inner Brooklyn (cluster 0).  Peripheral clusters with smaller counts represent niche markets (e.g., beach communities, Staten Island).  Pricing and marketing strategies should account for the different demand dynamics and regulatory environments in these areas.  For example, high‑priced Manhattan listings (cluster 2 from K‑Means) align spatially with the core DBSCAN cluster.
* **Regulatory monitoring:**  DBSCAN’s identification of peripheral clusters allows city officials or platform managers to monitor compliance and supply in outlying neighbourhoods where regulations may differ or enforcement is more challenging.

## Decision Rationale

**Why K‑Means?**  K‑Means is a fast, memory‑efficient clustering algorithm that handles continuous variables.  We considered hierarchical clustering, but its quadratic complexity (proportional to the square of the number of records) would be prohibitive for 21 000 listings.  We evaluated cluster counts via silhouette scores and selected k=4 for interpretability despite a higher silhouette at k=2, balancing granularity and cohesion.  Alternate cluster numbers (k=3 or k=5) yielded similar insights but with less distinct segments.

**Why DBSCAN?**  DBSCAN identifies arbitrarily shaped clusters and a noise set, making it suitable for geographic data where density varies.  K‑Means would impose spherical clusters and fail to isolate elongated neighbourhoods or sparsely populated areas.  Parameter tuning (`eps`, `min_samples`) required trade‑offs between cluster granularity and noise; we chose a moderate eps=0.2 to obtain meaningful neighbourhood groupings without excessive fragmentation.

## Requirements Recap

In this chunk we applied unsupervised clustering to the NYC Airbnb dataset.  K‑Means on standardized numeric features revealed four distinct listing profiles—luxury Manhattan, active mid‑range, typical and dormant—informed by price, host activity and review recency.  DBSCAN on lat–lon coordinates uncovered nine spatial clusters corresponding to the Manhattan/Brooklyn core and peripheral neighbourhoods (Far Rockaway, Staten Island, Coney Island)【788864212863535†L40-L44】.  These insights enable targeted pricing, marketing strategies and regulatory monitoring.

## Mindmap & Next Steps

- **Chunks 0–11**: Completed charter, EDA, data cleaning, feature engineering, feature selection, regression modelling and tree ensembles.
- **Chunk 12 – *this chunk***: Conducted clustering analyses to reveal behavioural and spatial segments.
- **Next**: **Chunk 13** will synthesise findings from EDA, modelling and clustering into actionable business recommendations for hosts and policymakers, summarise methodology limitations and provide reproducible code/environment details.

## Compute Notes

* **Sampling for silhouette & plotting:**  Silhouette evaluation used a 5 000‑row sample to reduce runtime; full K‑Means was run on the entire dataset.  The DBSCAN plot shows a random sample of 5 000 listings for clarity.
* **Memory usage:**  Standardizing features and fitting K‑Means and DBSCAN consumed under 2 GB of RAM.  Clustering operations completed within the per‑chunk runtime target.
* **Reproducibility:**  Random seeds were fixed for sampling and K‑Means initialization.  DBSCAN results may vary with parameter choice; the selected `eps` and `min_samples` values are documented above.
