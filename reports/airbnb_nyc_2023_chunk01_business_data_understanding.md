# CHUNK 1 – Business & Data Understanding

## High‑level business context

Airbnb has transformed the hospitality sector by enabling individuals to rent private rooms or entire homes to visitors.  New York City (NYC) consistently ranks among Airbnb’s largest markets due to high tourist demand and limited hotel capacity.  However, the city has also tightened regulation: **Local Law 18**, adopted in January 2022, requires hosts to register their short‑term rentals and prohibits booking platforms from processing transactions for unregistered listings.  Enforcement began on **September 5 2023**【223133324145814†L61-L70】, meaning only registered hosts may accept stays under 30 days.  The law and continuing debates about Airbnb’s impact on housing create a need for data‑driven insights about pricing and market behaviour.

## Dataset overview

### Choice of dataset and its coverage

We attempted to obtain the 2023 snapshot of Inside Airbnb’s detailed listings.  Direct download links to `data.insideairbnb.com` required authentication, and the 2023 summary data on Comparebnb was inaccessible.  Instead we found a **6 MB summary file `listings.csv`** already present in our environment (36 403 rows × 18 columns).  The file structure matches the Inside Airbnb summary schema (ID, name, host ID, host name, neighbourhood group, neighbourhood, latitude, longitude, room type, price, minimum nights, number of reviews, last review date, reviews per month, host listings count, availability 365, number of reviews in last 12 months, licence).  Inspection of the `last_review` field shows dates through **2 Aug 2025**, implying this dataset is a snapshot taken **Aug 2025** rather than 2023.  Because an accessible 2023 file was unavailable, we will treat this as the *nearest available snapshot* and note any implications.  We considered using the `nyc_listings.csv` file (38 792 rows) but chose `listings.csv` because it retains separate latitude/longitude columns (better for geospatial analysis) and aligns with Inside Airbnb’s data dictionary.  This decision trades recency (2025 vs 2023) for data completeness and ease of use.

### Schema and sample rows

A glance at the top rows reveals the key fields used for pricing models.  Below is a sample of selected columns:

| id   | neighbourhood_group | neighbourhood  | room_type     | price | minimum_nights |
|-----:|--------------------:|---------------:|---------------|------:|---------------:|
| 2539 | Manhattan           | Harlem         | Entire home/apt | 149  | 30 |
| 2595 | Manhattan           | Midtown        | Entire home/apt | 150  | 30 |
| 6848 | Brooklyn            | Williamsburg   | Entire home/apt | 81   | 30 |
| 6872 | Manhattan           | East Harlem    | Private room   | 65   | 30 |
| 6990 | Manhattan           | East Harlem    | Private room   | 62   | 30 |

This summary file lacks many rich attributes found in the full Inside Airbnb dataset (e.g., amenities, host experience, property type).  Consequently we will engineer additional features from the existing columns (e.g., neighbourhood effects, review counts per month and availability).  Key numerical fields (`price`, `minimum_nights`, `number_of_reviews`, `reviews_per_month`, `availability_365`) will require dtype down‑casting (`float32`/`int32`) and cleaning (e.g., ensuring `price` is numeric and removing currency symbols) during preprocessing.

### Basic descriptive statistics

Preliminary statistics (computed on selected columns) provide a sense of the price distribution and listing composition:

| Statistic | Count | Mean | Std Dev | Min | 25th pct | Median | 75th pct | Max |
|---------:|------:|-----:|-------:|----:|--------:|-------:|--------:|-----:|
| **Price (USD)** | 21 279 | 447.9 | 3 174.2 | 3 | 90 | 150 | 257 | 50 052 |

The median nightly rate is around \$150 with a skewed distribution (note the maximum above \$50 k).  A quarter of listings charge less than \$90 per night, and only 25 % exceed \$257.  In terms of accommodation types:

- **Room type counts:** Entire home/apartment (19 328 listings) > Private rooms (16 469) > Hotel rooms (378) > Shared rooms (228).
- **Neighbourhood groups:** Manhattan (16 225 listings), Brooklyn (13 322), Queens (5 336), Bronx (1 155) and Staten Island (365).
- **Review recency:** about 25 093 listings (≈69 %) have a `last_review` date; others have no recorded reviews (possible new or inactive listings).

These patterns suggest a highly skewed price distribution and a majority of entire home/apartment rentals.  Manhattan and Brooklyn together host about 80 % of listings.  Such imbalances must be considered when selecting validation strategies (e.g., stratifying by neighbourhood group and price bins).

## Known biases and limitations

Data scraped from Airbnb and shared by Inside Airbnb has notable limitations:

- **Snapshot bias.** Inside Airbnb’s datasets are snapshots of listings at a specific date and do not represent continuous activity.  The registration law enforcement in Sept 2023 removed many listings, so our August 2025 snapshot may include only compliant hosts.  Researchers from Cornell note that price data “is valid for the one day the data was scraped”【34995615265008†L774-L781】; time‑series analysis requires multiple snapshots, which are unavailable here.
- **Location anonymization.** Airbnb anonymizes listing coordinates by moving them up to 150 m from the actual address【788864212863535†L40-L45】.  This geospatial jitter could blur fine‑scale location effects but still allows neighbourhood‑level analysis.
- **Calendar uncertainty.** The availability calendar does not distinguish between booked nights and nights blocked by the host, so calculated availability likely understates occupancy【788864212863535†L45-L51】.  Hosts may also fail to keep the calendar updated【788864212863535†L51-L53】.
- **Scope.** Our data covers NYC only【34995615265008†L783-L784】.  Insights may not generalise to other cities.
- **Limited features.** The summary file lacks detailed attributes (amenities, host response rates).  Without these, model explanatory power might be limited; we will engineer proxy features (e.g., review counts per month) but recognise the constraint.

When interpreting results, we will highlight these biases and avoid over‑generalising.  Additionally, because Airbnb’s location data is anonymised and hosts may vary pricing over time, predictions should be viewed as indicative rather than exact.

## Decision rationale

- **Dataset selection.** We evaluated two candidate files: `listings.csv` (36 k rows, 18 columns) and `nyc_listings.csv` (38 k rows, 17 columns).  We selected `listings.csv` because it retains separate latitude/longitude fields (essential for geospatial clustering) and aligns with Inside Airbnb’s official summary schema.  We attempted to download the 2023 detailed dataset but encountered access restrictions; thus, the 2025 snapshot is used as the nearest available option.  While this reduces temporal alignment with 2023, it provides a complete and accessible dataset for modelling.
- **Statistical baseline.** A quick check of the price distribution confirmed a heavy right skew, which suggests that using a log transformation during modelling may stabilise variance.  We will explore both raw and `log1p(price)` targets later.
- **Simplification vs. completeness.** The summary dataset omits many features but reduces file size to ~6 MB, making it manageable within the 12 GB RAM limit.  Loading the full 75‑column detailed dataset (if accessible) would have required additional memory and processing time; given constraints, we accept the trade‑off.

## Requirements recap

This chunk introduced the business context (NYC Airbnb market and Local Law 18) and inspected the available dataset.  We summarised the schema, sample rows, and key descriptive statistics, and discussed data limitations such as snapshot bias, anonymised locations and calendar uncertainty【788864212863535†L45-L51】.  We documented the choice of dataset (`listings.csv` as the nearest accessible snapshot) and justified it based on data completeness and compute constraints.  

## Mindmap & where we are

```
Project (Airbnb NYC 2023/25)
├── Chunk 0: Project charter & plan ✔
└── Chunk 1: Business & data understanding (current)
    ├── Summarised NYC Airbnb market and regulation (Local Law 18)【223133324145814†L61-L70】
    ├── Selected summary dataset (listings.csv) and provided schema overview
    ├── Computed basic descriptive statistics (price distribution, room types)
    ├── Identified data biases and limitations【788864212863535†L40-L51】【34995615265008†L774-L781】
    └── Next: Data access & schema audit (Chunk 2) – load dataset with memory-aware dtypes, audit missingness
```

## Compute notes

- **Selective loading.** For this chunk we loaded only the summary `listings.csv` (≈36 k rows) and restricted to a subset of columns when computing statistics, ensuring minimal memory usage (< 50 MB).  The file fits comfortably in memory; in subsequent chunks we will downcast numeric types (`float32`, `int32`) and convert categorical variables to `category` to reduce memory further.
- **Parsing dates.** We parsed the `last_review` column as `datetime` when exploring review recency, but we will avoid parsing large text fields until necessary.
- **No heavy I/O.** We refrained from loading optional datasets (`reviews.csv`, `calendar.csv`) or detailed `listings` file due to access constraints and memory considerations.  If compute allows later, we may sample a few rows from `calendar.csv` for temporal analysis.

