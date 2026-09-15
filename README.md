# San Francisco Crime Analysis

An exploratory data analysis of San Francisco Police Department incident report data, using `pandas`, `sqlite3`, and `seaborn` to clean the data, run SQL queries for trend analysis, and visualize crime patterns by year, category, district, neighborhood, and day of week.

## Overview

This project pulls the City of San Francisco's public "Police Department Incident Reports: 2018 to Present" dataset directly from the [DataSF open data portal](https://data.sfgov.org), cleans and loads it into a local SQLite database, and runs a series of SQL queries to surface trends in crime volume, category, location, and resolution outcomes. Results are visualized with `matplotlib`/`seaborn`.

## Data Source

- **Dataset:** SFPD Incident Reports (2018–Present)
- **URL:** `https://data.sfgov.org/api/views/wg3w-h783/rows.csv?accessType=DOWNLOAD`
- The notebook downloads this CSV live at runtime — no local data file is required, but an internet connection is.

## Project Structure

```
.
├── crime-analysis.ipynb   # Main analysis notebook
├── crimes.db              # SQLite database created/overwritten on run
├── report.md              # Write-up of methodology and findings
└── README.md
```

## Requirements

- Python 3.x
- `pandas`
- `numpy`
- `matplotlib`
- `seaborn`
- `sqlite3` (Python standard library)
- Jupyter Notebook / JupyterLab

Install dependencies with:

```bash
pip install pandas numpy matplotlib seaborn jupyter
```

## How to Run

1. Clone or download this repository.
2. Launch Jupyter and open `crime-analysis.ipynb`.
3. Run all cells top to bottom. The notebook will:
   - Download the latest SFPD incident dataset from DataSF
   - Clean column names, parse date fields, and handle nulls
   - Load the cleaned data into a local `crimes.db` SQLite database
   - Run SQL aggregation queries (crimes per year, by category, by district, by neighborhood, by day of week, arrest vs. non-arrest, category by district)
   - Generate seaborn visualizations for each query
   - Print a summary of key findings in the final markdown cell

## Notebook Sections

1. **Importing Modules** — loads required libraries
2. **Cleaning the Data** — standardizes column names, parses datetime fields, identifies and handles nulls, checks/removes duplicates
3. **SQL Queries — Finding Trends** — loads data into SQLite and runs aggregation queries for yearly totals, category counts, district counts, neighborhood counts, arrest resolution, day-of-week counts, and category-by-district crosstabs
4. **Seaborn Graphs and Plots** — line, bar, pie, and heatmap visualizations of the query results
5. **Predictions and Analysis** — narrative summary of findings

See `report.md` for the full write-up of methods and results.

## Notes / Known Limitations

- Data cleaning calls (`crimes.dropna()`, `crimes.drop_duplicates(...)`) are run without reassignment, so they do not actually modify `crimes` in place — the underlying dataset used for analysis still contains nulls and any duplicate rows.
- 2025 (and the most recent period generally) is likely undercounted since not all incidents have been reported/recorded yet.
- The arrest/non-arrest classification only accounts for four of the possible `resolution` values; other resolution types are excluded from that query.