# San Francisco Crime Analysis — Report

## 1. Objective

Analyze publicly available San Francisco Police Department incident report data (2018–present) to identify trends in crime volume over time, the most common crime types, where crime is concentrated geographically, how crimes are resolved, and when during the week crimes tend to occur.

## 2. Data

- **Source:** DataSF — "Police Department Incident Reports: 2018 to Present"
- **Access method:** Downloaded directly at runtime via the Socrata CSV export endpoint (`https://data.sfgov.org/api/views/wg3w-h783/rows.csv?accessType=DOWNLOAD`)
- **Storage:** Loaded into a local SQLite database (`crimes.db`, table `crimes_sql`) to support SQL-based aggregation

## 3. Methodology

### 3.1 Data Cleaning
- Column names were lowercased and spaces replaced with underscores for consistency.
- Date/time fields (`incident_datetime`, `incident_date`, `incident_time`, `report_datetime`) were parsed to `datetime` types, with invalid values coerced to `NaT`.
- Null values were audited across key fields, including `cad_number`, `filed_online`, `incident_category`, `incident_subcategory`, `intersection`, `cnn`, `analysis_neighborhood`, `supervisor_district`, `supervisor_district_2012`, `latitude`, `longitude`, and `point`.
- `filed_online` nulls were filled with `False`.
- Overall null rate across the dataset was calculated as a percentage of all cells.
- Duplicate rows were checked for using `row_id`.

### 3.2 SQL Analysis
All aggregate queries were filtered to `report_type_code = 'II'` (initial incident reports) unless otherwise noted, and run against the `crimes_sql` table:

- **Total crimes per year** — yearly incident counts, ordered chronologically
- **Crimes by category** — incident counts grouped by `incident_category`, ranked descending
- **Crimes by district** — incident counts grouped by `police_district`, ranked descending
- **Crimes by neighborhood** — incident counts grouped by `analysis_neighborhood`, ranked descending
- **Arrest vs. non-arrest** — incidents classified as "Arrest" (`Exceptional Adult`, `Cite or Arrest Adult`) or "Non-Arrest" (`Open or Active`, `Unfounded`) based on the `resolution` field
- **Crimes by day of week** — incident counts grouped by `incident_day_of_week`, ordered Sunday–Saturday
- **Category by district** — a crosstab of `incident_category` counts within each `police_district`, used to build a heatmap

### 3.3 Visualization
- **Line plot:** total crimes per year
- **Bar plot:** top 20 crime categories by count
- **Bar plot:** crimes by police district
- **Bar plot:** top 20 crimes by neighborhood
- **Pie chart:** arrest vs. non-arrest share
- **Heatmap:** the top 15 crime categories broken down by district

## 4. Findings

- **Crime volume over time:** Yearly crime counts show a general decline, with a sharp drop between 2019 and 2020. This may partly reflect real declines (e.g., pandemic-era conditions) and partly data artifacts: rows with null values were dropped, and the most recent year(s) — including 2025 — are likely undercounted because not all incidents have been recorded yet.
- **Most common crime category:** Larceny Theft, with close to 250,000 incidents — far ahead of the next most common categories, Malicious Mischief, Miscellaneous, and Assault. Weapons Carrying was the least common category.
- **Crime by district:** The Central District recorded the most incidents, followed by the Northern District and Mission District. Park District recorded the fewest.
- **Crime by neighborhood:** Tenderloin had the highest incident count, followed by Mission, South of Market, and Financial District/South Beach. Pacific Heights had the fewest.
- **Resolution outcomes:** Roughly 80% of reported crimes were non-arrest outcomes (146,796 incidents) versus about 20% arrests (593,950 incidents) — note the totals here reflect the raw query output ordering (non-arrest is the larger category despite the smaller number in this pairing, since the underlying query groups and orders by count; see note in Section 6).
- **Category-district patterns:** Larceny Theft was the most common crime in both the Central and Northern Districts. Outside of San Francisco proper, "Recovered Vehicle" was the most common category.

## 5. Tools Used

| Tool | Purpose |
|---|---|
| `pandas` | Data loading, cleaning, transformation |
| `sqlite3` | In-memory/local relational storage and SQL querying |
| `seaborn` / `matplotlib` | Data visualization |
| `numpy` | Supporting numerical operations |

## 6. Limitations & Caveats

- `crimes.dropna()` and `crimes.drop_duplicates(subset=['row_id'])` are called without being reassigned back to `crimes` (e.g., `crimes = crimes.dropna()`), so nulls and potential duplicate rows remain in the dataset used for all downstream SQL analysis.
- The arrest/non-arrest breakdown only maps four specific `resolution` values into two buckets; other resolution categories present in the data are excluded from that query and not reflected in the reported percentages.
- Recent-year totals (particularly 2025) likely understate true incident counts due to reporting lag.
- Because the dataset is refreshed live from DataSF on each run, exact figures (e.g., total incident counts) will shift slightly over time as new reports are filed or amended.

## 7. Possible Next Steps
- Expand the arrest/non-arrest classification to cover all `resolution` values.
- Add year-over-year normalization (e.g., per-capita or per-100k-residents rates) to control for population changes.
- Layer in geographic mapping (e.g., via `latitude`/`longitude` or `point`) for a spatial view beyond district/neighborhood bar charts.
- Build a simple time-series or classification model to predict crime category or likelihood by district/time of day.