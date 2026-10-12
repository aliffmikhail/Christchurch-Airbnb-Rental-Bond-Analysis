# Christchurch Airbnb and Rental Market Analysis

A reproducible data-wrangling and analysis project combining monthly Inside Airbnb listings for Christchurch, New Zealand, with quarterly rental bond statistics from Tenancy Services. The project prepares and validates data, enriches Airbnb observations with Stats NZ Statistical Area 2 (SA2) geography, and generates summary statistics and visualisations. Where reporting periods and geographic identifiers can be matched, it compares short-term Airbnb prices with long-term rental prices.

## Team

| Team member | Course |
| ----------- | ------ |
| Dron Vihang Dalvi | DATA422 |
| Abdurrahman Rais Fadhil | DATA422 |
| Muhammad Aliff Mikhail Bin Norkamarulazhar | DATA201 |
| Asfa Hurin Binti Asmawi | DATA201 |

## Project overview

The project contains two related workflows:
1. **Airbnb workflow:** Prepares monthly source files, filters Christchurch City observations, combines and cleans monthly data, updates geographic mappings, and regenerates analysis outputs.
2. **Rental bond comparison:** Cleans quarterly rental bond data and compares it with Airbnb observations only for periods and area that can be matched.

The automated Airbnb workflow currently covers **October 2025** to **August 2026**.

## Quick start

### Requirements

- Python 3
- The Python packages imported by the project scripts
- A Koordinates API key when new or unresolved coordinates require geographic lookup

### Prepare the data

Place monthly Inside Airbnb New Zealand `listings.csv` downloads in `data/raw/`, using the naming convention `listings_YYYY_MM.csv`. For example:

`data/raw/listings_2026_07.csv`

`data/raw/listings_2026_08.csv`

Use the CSV file, not the compressed `csv.gz` file. Keep raw downloads unchanged to support reproducibility.

Store your Koordinates API key in a local `.env` file at the repository root:

`KOORDINATES_API_KEY=YOUR_API_KEY`

Do not commit `.env` or credentials to Git. Each team member who needs to run geographic lookups should configure their own key.

Scrape dates used to calculate `days_since_last_review` are maintained in `config/airbnb_scrape_dates.csv`. Add the corresponding scrape date whenever a new monthly dataset is added.

### Run the pipeline

From the repository root:

`python scripts/run_airbnb_pipeline.py`

The orchestrator runs stages in sequence and stops if a stage fails, preventing later steps from using incomplete or invalid outputs.

## Automated workflow

1. **Prepare monthly data**: Discover available monthly source files, select Christchurch City observations, and add `month_year`.
2. **Combine monthly data**: Concatenate processed monthly file into one Christchurch dataset.
3. **Clean and validate**: Apply cleaning rules and generate the cleaned combined dataset and audit information.
4. **Update geographic mappings**: Reuse the saved coordinate-to-SA2 lookup and query Koordinates only for new or unresolved coordinate pairs.
5. **Run the rental bond comparison**: Use only reporting periods supported by the available bond dataset.
6. **Generate summary statistics**.
7. **Regenerate visualisations and missing-value summaries**.

## Data sources

### Airbnb listings

- **Provider:** Inside Airbnb, an independent, non-commercial open-data initiative
- **Source:** https://insideairbnb.com/get-the-data/
- **File:** `listings.csv` (CSV, not `.csv.gz`)
- **Project coverage:** Christchurch City listings from monthly New Zealand snapshots, October 2025-August 2026

#### Data Dictionary

| Column Num | Column | Data Type | Description |
| ----- | ----- | ----- | ----- |
| 1 | id | Integer/String | Unique identifier assigned by Airbnb for the listing. |
| 2 | name | String | Public name or title of the listing as defined by the host. |
| 3 | host_id | Integer | Unique identifier for the host user. |
| 4 | host_name | String | First name (or display name) of the host. |
| 5 | neighbourhood_group | String | Broader region, borough, or municipality group (may be blank/null for some areas). |
| 6 | neighbourhood | String | Specific neighborhood, district, or local area of the listing. |
| 7 | latitude | Float | World Geodetic System (WGS84) latitude coordinate of the listing. |
| 8 | longitude | Float | World Geodetic System (WGS84) longitude coordinate of the listing. |
| 9 | room_type | String | Categorization of the space offered (Entire home/apt, Private room, Shared room, or Hotel room). |
| 10 | price | Numeric / Currency | Nightly rental price (in local currency). |
| 11 | minimum_nights | Integer | Minimum required length of stay in nights. |
| 12 | number_of_reviews | Integer | Total cumulative count of guest reviews received by the listing. |
| 13 | last_review | Date (YYYY-MM-DD) | Date of the most recent guest review.|
| 14 | reviews_per_month | Float | Average calculated number of reviews received per month over the listing's lifetime. |
| 15 | calculated_host_listings_count | Integer | Total number of listings managed by the host in this region. |
| 16 | availability_365 | Integer | Total number of days the listing is available for booking within the next 365 days. |
| 17 | number_of_reviews_ltm | Integer | Number of reviews received by the listing within the last 12 months. |
| 18 | license | String | Local permit, registration, or business license number (if required/provided). |

Some fields may contain missing values. Airbnb coordinates are masked for privacy and may not identify the exact property location.

### Rental bond statistics

- **Provider:** Tenancy Services, New Zealand
- **Dataset:** Detailed rental bond data
- **Source:** https://www.tenancy.govt.nz/about-tenancy-services/data-and-statistics/rental-bond-data/
- **Raw file:** `data/raw/rental_bond_detailed_2020_2026.csv`

#### Data Dictionary

| Column Num | Column | Data Type | Working Description |
| ----- | ----- | ----- | ----- |
| 1 | TimeFrame | string | Reporting reference start date for the quarterly bond summary (e.g., 2020-01-01 to 2026-04-01). |
| 2 | Location Id | float64 | Statistical Area 2 (SA2 2019) geographic code (includes special code -99 for unassigned locations). |
| 3 | Dwelling Type | string | Property structure category (ALL, Apartment, Boarding House, Flat, House, Room). |
| 4 | Number Of Beds | string | Bedroom count per listing (1, 2, 3, 4, 5, 5+, ALL, or missing/null). |
| 5 | Total Bonds | int64 | Total number of tenancy bonds lodged with Tenancy Services during the reference period. |
| 6 | Active Bonds | int64 | Cumulative count of active bonds held at the end of the reference period. |
| 7 | Closed Bonds | int64 | Total number of tenancy bonds refunded/closed during the reference period. |
| 8 | Median Rent | float64 | Median weekly rental price in NZD ($). |
| 9 | Geometric Mean Rent | float64 | Geometric mean weekly rental price in NZD ($), dampening the effect of extreme values. |
| 10 | Upper Quartile Rent | float64 | Upper quartile (75th percentile) weekly rental price in NZD ($). |
| 11 | Lower Quartile Rent | float64 | Lower quartile (25th percentile) weekly rental price in NZD ($). |
| 12 | Log Std Dev Weekly Rent | float64 | Standard deviation of log-transformed weekly rent, measuring price dispersion. |

The bond data are quarterly and use SA2-2019 geographic definitions.
Some bond counts are rounded and small categories may be suppressed by the provider.

### Geographic lookup

The project uses the Koordinates Query API to map Airbnb latitude/longitude coordinates to a Stats NZ SA2 area code. The API key is supplied locally through the `KOORDINATES_API_KEY` environment variable.

The Airbnb geographic field uses SA2-2026 codes, while the rental bond
source documentation refers to SA2-2019 geography. This difference can
contribute to unmatched records.

## Current data coverage

### Airbnb

The automated Airbnb workflow currently covers **October 2025 through August 2026**. The current generated outputs report:
- Combined input: **35,796 listing-month observations**
- Cleaned combined dataset: **24,414 listing-month observations**
- Cleaned dataset width: **17 columns**
  
These are run-specific figures and may change when source data or
cleaning rules are updated.

### Rental bond data

The cleaned bond dataset currently contains reporting periods:
- `2025-10-01`
- `2026-01-01`
- `2026-04-01`

These periods support the comparison through June 2026. July and August
2026 remain available for Airbnb-only analyses; the workflow does not
invent or carry forward bond values for unavailable periods.

## Cleaning and validation

### Airbnb cleaning

The cleaning stage prepares the combined Christchurch dataset for downstream analysis. The current rules include:
- Remove rows with missing prices because price is required for pricing analyses.
- Exclude nightly prices above NZ$1,500 as extreme values outside the intended analytical scope.
- Retain `month_year` to identify the source month for each observation.
- Fill missing `reviews_per_month` values with `0.0` where the listing has no reviews.
- Fill missing `minimum_nights` values using the observed dataset median.
- Represent missing host names as `"Unknown"`.
- Remove `license` and `neighbourhood_group` where they provide no useful variation for the Christchurch analysis.
- Check required columns and sanity conditions, and report rows removed and retained.

These rules affect the analysis and should be reviewed if the research
questions or source schema change.

### Rental bond cleaning

The bond-cleaning workflow filters the source to the reporting periods used in the project, checks missing and invalid values, audits duplicates and rent-statistic consistency, and excludes records whose geographic identifiers cannot be mapped reliably for the analysis.
Missing bedroom categories are retained rather than imputed, and genuine extreme rental values are not automatically removed.

### Pipeline validation

The automated workflow checks required columns, cleaning outputs, coordinate bounds, duplicate spatial lookup keys, spatial lookup match rate, missing area codes, preservation of Airbnb row counts during geographic enrichment, valid month values, availability of matching bond periods, and missing scrape-date metadata. Critical validation failures stop the pipeline rather than silently passing inconsistent data downstream.

## Geographic enrichment and caching

The persistent coordinate lookup is stored in `data/processed/Christchurch_coordinate_area_lookup.csv`. The enriched dataset is stored in `data/processed/Christchurch_Airbnb_with_area_codes.csv`.

Previously resolved coordinate pairs are reused; only new or unresolved pairs require additional Koordinates API queries. The current run reported **3,927 unique coordinate pairs** required by the cleaned Airbnb dataset, all already available in the lookup, so **0 new API queries** were required. The lookup contained **4,095 coordinate pairs** at the time of that run. These counts may change as data are added.

## Airbnb and rental bond comparison

The integration analysis joins Airbnb observations to rental bond records using Airbnb `area_code` and bond `Location Id`, together with the aligned reporting period.

### Time alignment

| Airbnb observation months | Bond reporting period |
| ------------------------- | --------------------- |
| October-December 2025 | `2025-10-01` |
| January-March 2026 | `2026-01-01` |
| April-June 2026 | `2026-04-01` |

For the overall location-level comparison, the analysis uses bond records where `Dwelling Type = ALL` and `Number Of Beds = ALL`, avoiding the combination of overlapping subgroup medians.

Weekly median long-term rent is converted to a daily equivalent:

`Daily Rent = Median Rent / 7`

For matched observations:

`Rent Gap = Airbnb Nightly Price - Daily Long-Term Rent`

Outputs include area-level median and maximum price gaps, observation counts, Airbnb listing counts, and active bond counts. Active bonds represent bonds active at the end of the reporting period; they are not a count of vacant or currently available properties.

### Recorded integration results

The recorded historical integration run reported:
- Airbnb observations before join: **18,080**
- Observations after join: **18,080**
- Matched observations: **13,821**
- Unmatched observations: **4,259**
- Match rate: **76.4%**

The unchanged row count indicates that the join did not multiply Airbnb observations in that run. These figures refer to the historical October 2025-June 2026 dataset, not the current October 2025-August 2026 pipeline totals.

Recorded findings included:
- **Wigram West (SA2 322800):** largest median price gap, approximately NZ$243.64 per night, based on 6 observations.
- **Addington West (SA2 326100):** largest single observed price gap, NZ$1,435 per night, based on 201 observations.
- **Christchurch Central (SA2 326600):** median Airbnb nightly price of NZ$238 in the recorded cleaned dataset.

These are historical run results and may change after future data updates. Area-level results should be interpreted alongside observation counts.

## Main outputs

Generated datasets include:

`data/processed/christchurch_listings_combined.csv`

`data/processed/christchurch_listings_combined_cleaned.csv`

`data/processed/Christchurch_coordinate_area_lookup.csv`

`data/processed/Christchurch_Airbnb_with_area_codes.csv`
  
Summary statistics, plots, and missing-value visualisations are written to `outputs/`.

Further implementation notes are available in

`docs/deliverable7_automation.md`

## Reproducibility and limitations

- Keep raw downloads unchanged and record each monthly Airbnb scrape date.
- Never commit API keys or other credentials.
- Run the pipeline from the repository root so relative paths resolve correctly.
- Review validation messages and audit outputs after each run.
- Airbnb advertised nightly prices and long-term rental bond statistics describe different rental markets. Converting weekly rent to a daily value aligns units but does not make the markets equivalent.
- Account for the SA2-2026 versus SA2-2019 geographic-definition difference when interpreting unmatched records.
- Do not extend the bond comparison into periods without matching bond data.
- Interpret area-level results in light of their observation counts.
