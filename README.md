# NYC Airbnb 2023 Pricing Analysis

This repository contains the full CRISP‑DM analysis of the 2023 New York City Airbnb dataset.  The project was executed in a series of methodical chunks that explore the business question, perform exploratory data analysis, engineer features, build predictive models, and extract actionable insights.

## Project Overview

The objective of this project is to understand the factors that drive nightly prices for Airbnb listings in New York City and to build interpretable predictive models.  We follow the CRISP‑DM process:

1. **Project Charter & Business Understanding** – define objectives, assess data availability and propose a plan.
2. **Data Access & Schema Audit** – safely load the dataset and examine its structure and missingness.
3. **Exploratory Data Analysis (EDA)** – univariate, bivariate, temporal and geospatial analyses to reveal key patterns.
4. **Cleaning & Pre‑processing** – handle missing values, remove outliers and transform variables.
5. **Feature Engineering & Selection** – derive informative variables and select the most predictive ones.
6. **Modelling** – baseline regressions, regularized linear models and tree ensembles to predict log‑price.
7. **Clustering** – segment listings by behaviour and geography.
8. **Synthesis & Recommendations** – summarise findings, limitations and actionable recommendations for hosts and policymakers.

## Repository Structure

```
airbnb_nyc_2023_repository/
├── README.md                 ← this file
├── reports/                  ← Markdown reports for each analysis chunk
├── images/                   ← Plots and visualisations used in the reports
├── data/                     ← Processed data and summary tables
└── notebook/                 ← Jupyter notebook to reproduce the full analysis
```

### Reports

Each numbered report documents a specific step of the analysis and can be read sequentially:

| Chunk | File | Description |
|---|---|---|
| 00 | `airbnb_nyc_2023_chunk00_project_charter.md` | Project charter with objectives, CRISP‑DM plan and chunk schedule. |
| 01 | `airbnb_nyc_2023_chunk01_business_data_understanding.md` | Business context, regulatory background and dataset scope. |
| 02 | `airbnb_nyc_2023_chunk02_data_access_schema_audit.md` | Data loading strategy, schema inspection and missingness patterns. |
| 03 | `airbnb_nyc_2023_chunk03_eda_univariate.md` | Univariate exploration of price, room type, reviews and other features. |
| 04 | `airbnb_nyc_2023_chunk04_eda_bivariate_temporal.md` | Bivariate relationships, correlation matrix and temporal price trends. |
| 05 | `airbnb_nyc_2023_chunk05_geo_eda.md` | Geospatial analysis using hexbin maps to show listing density and price variation. |
| 06 | `airbnb_nyc_2023_chunk06_cleaning_preprocessing.md` | Data cleaning, handling of missing values, imputation and transformation. |
| 07 | `airbnb_nyc_2023_chunk07_outlier_analysis.md` | Outlier detection and capping strategy for price and minimum nights. |
| 08 | `airbnb_nyc_2023_chunk08_feature_engineering.md` | Creation of new features (e.g. log transforms, bins) and winsorisation. |
| 09 | `airbnb_nyc_2023_chunk09_feature_selection.md` | Evaluation of features via correlation, mutual information and random forest importance. |
| 10 | `airbnb_nyc_2023_chunk10_modeling_linear.md` | Baseline and linear regression models; comparison of raw vs log‑price targets. |
| 11 | `airbnb_nyc_2023_chunk11_tree_ensembles.md` | Non‑linear models (Random Forest, Gradient Boosting) and feature importances. |
| 12 | `airbnb_nyc_2023_chunk12_clustering.md` | K‑Means and DBSCAN clustering to segment listings and neighbourhoods. |
| 13 | `airbnb_nyc_2023_chunk13_final_synthesis.md` | Final summary of insights, model performance, recommendations and limitations. |
| — | `airbnb_nyc_2023_medium_story.md` | A narrative version suitable for a Medium article, weaving together the key findings and plots. |

### Images

The `images/` folder contains all visualisations used throughout the reports. Notable plots include:

* **missingness_heatmap.png** – visualises missing values across columns.
* **price_log_price_hist.png** – shows raw and log‑transformed price distributions.
* **room_type_distribution.png** and **neighbourhood_group_distribution.png** – distribution of key categorical variables.
* **correlation_matrix.png** and **bivariate_scatter.png** – bivariate relationships.
* **price_trend_year.png** – temporal trend of prices based on review dates.
* **listing_density_hexbin.png** and **average_price_hexbin.png** – spatial patterns of listing density and average prices.
* **rf_feature_importances_bar.png** – top features identified by tree ensembles.
* **pred_vs_actual_log.png** and **residuals_vs_pred_log.png** – diagnostic plots for the linear models.
* **dbscan_geo_clusters_plot.png** – geographic clusters identified via DBSCAN.

### Data

Processed datasets and summary tables are stored in the `data/` folder. Key files:

| File | Description |
|---|---|
| `listings_processed.csv` | Cleaned listing‑level data after handling missing values and outlier capping. |
| `listings_feature_engineered.csv` | Dataset with engineered features (log transforms, bins, winsorised price). |
| `listings_schema_summary.csv` | Column‑wise summary of the original schema (dtype, missingness, uniqueness). |
| `kmeans_cluster_summary.csv` | Summary statistics for each behavioural cluster. |
| `dbscan_geo_clusters.csv` | Assigned DBSCAN cluster label for each listing. |
| `dbscan_geo_cluster_price_summary.csv` | Price statistics per geographic cluster. |
| `feature_selection_summary.csv` | Rankings of features by correlation, mutual information and random‑forest importance. |
| `rf_feature_importance_top20.csv` | Top 20 features by Random Forest importance. |
| `chunk10_linear_metrics.csv` | Evaluation metrics for baseline and linear models (RMSE, MAE, R²). |

### Notebook

The Jupyter notebook in `notebook/airbnb_nyc_2023_analysis.ipynb` reproduces the full analysis end‑to‑end.  It contains code cells interspersed with explanatory markdown so you can re‑run the study or adapt it to other cities or snapshots.

## Getting Started

1. Clone or download this repository.
2. Install the required Python libraries (see `requirements.txt` if provided, or use Anaconda with pandas, numpy, seaborn, scikit‑learn, etc.).
3. Launch the notebook in Jupyter or Google Colab.
4. Explore the reports in the `reports/` directory for a narrative of each stage.
5. Use the processed datasets in `data/` for further modelling or analysis.

## Notes

* All analysis uses the 2023 NYC Airbnb snapshot scraped from Inside Airbnb, which anonymises coordinates within 150 m of the true location for privacy.
* The pipeline is designed to be reproducible under ~12 GB of RAM, with careful dtype downcasting and sampling.
* For more context on connecting GitHub to ChatGPT, see the [OpenAI help article](https://help.openai.com/en/articles/11145903-connecting-github-to-chatgpt).
