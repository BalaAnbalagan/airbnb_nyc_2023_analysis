# Airbnb NYC 2023 Project Charter & Plan (Chunk 0)

## 1 Dataset availability

The objective is to predict the nightly price of New York City Airbnbs using the Inside Airbnb data.  A comprehensive **Kaggle/Inside Airbnb data card** titled *“New York City Airbnb 2023, Public Data”* indicates that NYC Airbnb data are available from **2011 through 2023** and that the dataset “describes the listing activity and metrics in NYC for 2023”【247656804193548†screenshot】.  This confirms that we can access at least one snapshot of the 2023 listings.  Should the 2023 full detail file be unavailable in the Inside Airbnb archive, we will use the nearest available snapshot (e.g., early 2024 or late 2022) and explicitly document the substitution.  

Inside Airbnb’s download page lists datasets for each city with **summary and detailed files**.  For New York City (August 1, 2025 snapshot) the available files include `listings.csv.gz` (detailed listings), `calendar.csv.gz` (daily price/availability), `reviews.csv.gz` (review-level data) and summary versions of these files【580732287685971†L1469-L1487】.  We will prioritize loading the 2023 `listings.csv` (or the nearest file) because it contains listing‑level attributes (host status, location, number of reviews, room type, pricing) necessary for modeling.  We will optionally sample from `reviews.csv` and `calendar.csv` if resources permit.  

## 2 Business understanding

**Problem statement / business question.**  
NYC’s tourism sector is a major driver of short‑term rentals.  Hosts need to price their listings competitively while accounting for market conditions, amenities and neighbourhood effects.  The aim is to **identify factors driving nightly Airbnb prices in New York City** and to **develop a predictive model** to help hosts set realistic base prices.  This requires understanding how features such as room type, neighbourhood, number of guests, amenities, host experience and review patterns influence price.  The project will deliver actionable insights: e.g., which amenities yield higher returns, how location and seasonality affect price, and recommendations for pricing strategies.

## 3 CRISP‑DM roadmap tailored to this problem

The Cross‑Industry Standard Process for Data Mining (CRISP‑DM) provides a structured methodology.  We will follow it sequentially while including decision rationales and compute/memory considerations at each step:

1. **Business & data understanding** – articulate the pricing problem and evaluate the dataset scope (NYC 2023) and any limitations.
2. **Data access & schema audit** – safely load `listings.csv` with selective column types (e.g., `float32` for numerical fields and `category` for nominal fields) and map missingness.  Optional: sample `calendar.csv` (30–60 days) and `reviews.csv` for temporal/geographic analysis.
3. **Exploratory data analysis (EDA)**
   - *Univariate*: summarise distributions of price, room types, host features, number of reviews, etc.
   - *Bivariate & temporal*: explore price vs. other variables (e.g., price vs. room type, host response rate, number of guests); examine simple temporal signals if we include calendar data.
   - *Geo‑EDA*: if memory allows, aggregate prices by neighbourhood or hex bins and produce spatial visualisations to uncover geographic price patterns.
4. **Data cleaning & pre‑processing** – handle currency symbols, parse amenities, encode categorical features, impute missing values, normalise skewed variables and prepare data for modeling.
5. **Outlier analysis & processing** – identify extreme prices via IQR/robust z‑score and decide whether to winsorise, cap or remove them while justifying the impact on model performance vs. representativeness.
6. **Feature engineering** – derive new predictors such as amenity counts, sentiment length of descriptions, host tenure, review frequencies and geospatial clusters.
7. **Feature selection** – filter correlated/irrelevant features via correlation analysis, mutual information or model‑based selection; include a rationale for excluding features that might leak target information.
8. **Clustering** – perform unsupervised grouping of listings using K‑means (on scaled numeric features) or DBSCAN for latitude/longitude sampling; interpret clusters in business terms (e.g., luxury vs. budget segments).
9. **Modeling** – start with **baselines** (mean and median price) then fit linear models (OLS, Ridge, Lasso) using both raw `price` and `log1p(price)` targets; subsequently build tree‑based ensembles (Random Forest, Gradient Boosting) with appropriate hyperparameters.  Emphasise cross‑validation, hold‑out sets and evaluation metrics (RMSE, MAE, R²).  Explore partial dependence and, if feasible, SHAP explanations.
10. **Final synthesis** – compile findings into an executive summary with practical recommendations for hosts and policymakers, discuss limitations (e.g., data snapshot, regulation changes in 2023), and provide a reproducibility appendix (software versions, environment specifications, and run steps).

## 4 Chunk schedule (Standard mode)

To comply with the 12 GB RAM constraint and typical 3–8 minute per‑chunk runtime, we divide the work into manageable chunks and specify compute tactics.  Each chunk will have its own write‑up (`.md` file) saved with a consistent naming convention.

| Chunk | Focus | Key tasks & memory strategy | Target runtime |
|---|---|---|---|
| **0** | **Project charter & plan** | Confirm dataset availability; outline business problem, CRISP‑DM roadmap, chunk schedule, and file-saving conventions. *No large data loading.* | ≤ 5 min |
| **1** | **Business & data understanding** | Summarise high‑level features and known biases in Airbnb market; perform lightweight schema inspection using a head of `listings.csv` (restrict columns). | 5–8 min |
| **2** | **Data access & schema audit** | Load `listings.csv` with memory‑efficient dtypes; produce a schema table (column names, types, missingness); produce a missingness heatmap (sample if needed). | 6–8 min |
| **3** | **Univariate EDA** | Compute and visualise distributions for price and other key features; use histograms, box plots and summary stats. Downcast floats to `float32` and categories to `category` to manage memory. | 6–8 min |
| **4** | **Bivariate & temporal EDA** | Examine price vs. other variables using scatter plots and box plots; if calendar data is loaded, sample 30–60 days of price/time to study seasonal effects. | 6–8 min |
| **5** | **Geo‑EDA** | Load neighbourhood boundaries (geojson) and join with listings; compute average price per neighbourhood; produce a choropleth or hexbin map. Use sampling for high‑resolution lat/lon plotting. | 6–8 min |
| **6** | **Cleaning & pre‑processing** | Clean price field (remove currency symbols); convert amenities to counts; handle missing values; encode categorical variables (one‑hot or ordinal as appropriate). Document decisions on imputation and scaling. | 6–8 min |
| **7** | **Outlier analysis & processing** | Identify outliers via IQR or robust z‑scores; decide whether to cap or remove; compare model performance with and without outlier mitigation. | 6–8 min |
| **8** | **Feature engineering** | Create new features (amenity counts, host experience, review sentiment lengths, geospatial bins); ensure memory safety through vectorised operations; justify each engineered feature. | 7–8 min |
| **9** | **Feature selection** | Apply correlation filtering, mutual information, and model‑based methods (e.g., RFE with linear models); justify selected features; avoid leakage. | 6–8 min |
| **10** | **Modeling – baselines & linear models** | Establish naive baselines (mean/median price); fit linear regression, ridge and lasso models on raw and log‑transformed targets; evaluate via RMSE, MAE and R² on a hold‑out set. Document train/validation split strategy (e.g., 80/20 random split). | 7–8 min |
| **11** | **Modeling – tree ensembles & interpretability** | Train Random Forest and Gradient Boosting regressors; compare performance to linear models; compute feature importances; if feasible, sample data for SHAP values and partial dependence plots. | 7–8 min |
| **12** | **Clustering & segment analysis** | Cluster scaled numeric features with K‑means; optionally cluster geographic coordinates with DBSCAN; interpret clusters (e.g., budget vs. premium segments) and cross‑tab with neighbourhoods and amenity levels. | 7–8 min |
| **13** | **Final synthesis & recommendations** | Summarise results, limitations (e.g., changes in NYC short‑term rental regulations in Sept 2023), and actionable recommendations for hosts; produce a reproducibility appendix (versions, seeds, run steps). | 7–8 min |

*Note:* If compute budgets tighten, we will sample heavy tasks (e.g., only 30% of rows for geospatial clustering or SHAP) and downcast data types to `float32`/`int32`/`category` to reduce memory usage.

## 5 File‑saving conventions

- Each chunk’s write‑up will be saved in the `/home/oai/share/` directory using the pattern `airbnb_nyc_2023_chunkXX_<description>.md`, where `XX` is the zero‑padded chunk number.  Example: this file is `airbnb_nyc_2023_chunk00_project_charter.md`.
- When generating outputs like figures or tables, we will embed the code and textual insights within the report; heavy images will be saved only if necessary.
- At the end of the project, we will compile an executive summary and reproducibility appendix summarising all chunks.

## Requirements Recap

We must predict nightly Airbnb prices for New York City using the 2023 Inside Airbnb dataset (or the nearest snapshot).  Following the CRISP‑DM methodology, we will deliver a sequence of well‑structured chunks covering business understanding, data access, exploratory analysis, cleaning, feature engineering/selection, modeling (baselines → linear → tree ensembles), clustering, and final recommendations.  Each chunk will include a **decision rationale** explaining method choices and compute considerations, adhere to memory constraints (~12 GB RAM), and save results in appropriately named markdown files.

## Mindmap & Where We Are

```
Project (Airbnb NYC 2023)
├── Chunk 0: Project charter & plan (current)
│   ├── Confirm dataset availability (NYC 2011–2023)【247656804193548†screenshot】
│   ├── Define business problem & CRISP‑DM roadmap
│   ├── Outline chunk schedule & compute strategy
│   └── Establish file‑saving conventions
└── Next steps
    ├── Chunk 1: Business & data understanding – summarise data context and high‑level features
    ├── Chunk 2: Data access & schema audit – load listings, inspect schema, missingness
    ├── … follow the schedule above …
```

## Compute Notes

- **Memory safety:** We will load only necessary columns at each step and downcast numerical types to `float32`/`int32` and categorical variables to `category` to conserve memory.  Text fields like `amenities` will initially be kept as objects, then processed to counts to reduce memory footprint.
- **Sampling for heavy tasks:** For geospatial mapping and SHAP analysis, we will sample 30–40% of the rows or restrict to a single month of calendar data.  This reduces computational cost while preserving representative patterns.
- **Reproducibility:** Random seeds will be set consistently (e.g., `np.random.seed(42)`) for splitting and sampling.  We will document environment details (Python version, library versions) in the final reproducibility appendix.
- **Runtime management:** Each chunk is designed to run within 3–8 minutes.  If operations exceed this, we will either simplify (e.g., reduce features) or postpone heavy analysis to later chunks with sampling.

