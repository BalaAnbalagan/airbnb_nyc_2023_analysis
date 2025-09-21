# CHUNK 5 – Spatial (Geo) Exploratory Data Analysis

## Objectives

This section investigates the spatial distribution of NYC Airbnb listings and their prices.  Latitude and longitude are included in the dataset, but Airbnb anonymises each location by shifting it within a ~150 m circle to protect privacy【788864212863535†L40-L44】.  Consequently, maps should be interpreted at neighbourhood level rather than exact addresses.  We explore listing density, spatial variation in price via hexagonal binning, and identify high‑priced neighbourhoods.

## Listing Density

The hexbin plot below counts the number of listings in each hexagonal cell (grid size = 50).  Darker cells indicate denser areas.  Unsurprisingly, the densest clusters appear in **Manhattan** (especially Midtown, Lower Manhattan and the East/West Villages) and **Brooklyn** (Williamsburg, Bushwick and Bedford‑Stuyvesant).  Outer boroughs like Staten Island have sparse coverage.

![Listing density hexbin map]({{file:file-DVxGejgUrLMpHDKK654qDt}})

## Spatial Variation in Price

To visualise how price varies across the city, we aggregated price within the same hexagonal grid using the mean price (only bins with ≥5 listings to avoid unreliable estimates).  Hotter colours correspond to higher average nightly rates.  The highest‑priced areas are concentrated in Manhattan’s upscale neighbourhoods: **TriBeCa**, **SoHo**, **NoHo**, **Greenwich Village** and the **Financial District**.  Some pockets in Brooklyn (e.g., **Vinegar Hill** near the waterfront) also command premium rates.  By contrast, northern Manhattan, much of Queens and the Bronx show cooler colours, indicating lower average prices.

![Average price by location (hexbin)]({{file:file-WTCbFPPtZ7NsrqQrVjAcxN}})

## Top Neighbourhoods by Median Price

|Neighbourhood|Median Price (USD)|
|---|---|
|Riverdale|1 200.5|
|Fort Wadsworth|600.0|
|NoHo|525.0|
|Holliswood|485.0|
|Tribeca|462.0|
|Greenwich Village|390.5|
|SoHo|358.5|
|West Village|350.0|
|Theater District|350.0|
|Financial District|327.0|

These neighbourhoods are among the most expensive areas in NYC.  Some, like Riverdale and Fort Wadsworth, have relatively few listings, so their medians are influenced by small sample sizes.  The well‑known high‑end neighbourhoods (NoHo, Tribeca, SoHo, Greenwich Village) align with the heatmap observations.

## Interpretation & Insights

* **Dense clusters** in Manhattan and parts of Brooklyn reflect high tourist demand and a concentration of hosts.  These areas offer abundant hotel alternatives and attractions, so supply is abundant.
* **Premium zones** around Lower Manhattan (TriBeCa/SoHo/Financial District) and certain parts of Brooklyn coincide with affluent residential areas and business districts.  Guests pay a premium to stay near tourist attractions and commercial centers.  Conversely, listings in northern Manhattan, the Bronx and outer Queens are more affordable.
* **Data caveats** – Because Airbnb randomises coordinates within a 150‑metre radius【788864212863535†L40-L44】, the hexbin visualisations are appropriate for aggregated patterns but should not be used to pinpoint individual properties.  Additionally, some high‑priced neighbourhoods have small sample sizes; caution is needed when interpreting their medians.

## Decision Rationale

* **Hexbin vs. scatter plot** – A naive scatter plot of ~36 k points would lead to severe overplotting, making it impossible to discern spatial patterns.  Hexagonal binning summarises local density and allows aggregation of price statistics within each hex.  A grid size of 50 balances spatial resolution with stability (smaller bins yield noisier estimates).  We could have used kernel density estimation or Voronoi tessellation, but those methods are computationally heavier and less interpretable for audiences unfamiliar with advanced geospatial analytics.
* **Mean vs. median for spatial aggregation** – We computed the **mean** price per hexagon rather than the median to avoid zeros in bins with very few observations.  For neighbourhood‑level rankings we used the **median** to lessen the influence of extreme luxury listings.  An alternative would be to apply trimmed means or quantile regression surfaces; these were deemed unnecessary at this stage.
* **Neighbourhood table** – Limiting the table to the top 10 neighbourhoods simplifies the narrative.  The full list contains 187 neighbourhoods, many with very small counts.  Including them would clutter the table without delivering additional insight.

## Requirements Recap

This chunk delivered a **geospatial EDA** of NYC Airbnb listings.  We created hexbin maps to illustrate listing density and average price, identified high‑priced neighbourhoods, and discussed spatial patterns.  Decision rationales explained the choice of hexagonal bins, the use of mean versus median for spatial statistics, and the limitations due to location anonymisation.

## Mindmap & Where We Are

* **Bivariate & Temporal EDA** – Completed (Chunk 4).
* **Geospatial EDA** – Completed (this chunk).  We now understand how listings and prices vary across NYC.
* **Next steps** –  
  - **Cleaning & Pre‑processing** (Chunk 6): handle missing values, encode categorical variables, normalise continuous features and prepare the dataset for modelling.  
  - **Outlier Analysis & Processing** (Chunk 7): identify and handle extreme values (e.g., luxury hotel rooms) to improve model robustness.

## Compute Notes

* **Hexbin resolution** – A grid size of 50 yields ~2 500 cells, manageable for memory and computation.  For price aggregation we excluded bins with fewer than five listings to avoid unreliable averages.
* **Memory and CPU usage** – The geospatial analysis was performed on the full dataset (~36 k rows); the computations fit easily within the 12 GB limit (RAM footprint remained <10 MB).  No sampling was required.  All plots were saved at 150 dpi for clarity without excessive file size.
