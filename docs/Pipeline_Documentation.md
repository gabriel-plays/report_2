# Data Pipeline Documentation

*How raw SAPS crime statistics and police station GIS data become `data/stations.json` and `data/precincts.json`*

---

## Overview

The pipeline takes two independent source streams — SAPS crime statistics spreadsheets and police station shapefiles — and produces two pre-computed JSON files served directly by the static web app with no backend required.

```
source/crime_stats/*.xlsx  ──┐
                              ├──► pipeline/clean/   ──► pipeline/enriched/   ──► data/stations.json
source/police_boundaries/  ──┘                                                ──► data/precincts.json
```

---

## Stage 1 — Ingest & Clean (`pipeline/clean/`)

### Input files

| File | Description |
|------|-------------|
| `source/crime_stats/Crime-Statistics-2020_2021.xlsx` | Annual release |
| `source/crime_stats/Crime-Statistics-2021_2022.xlsx` | Annual release |
| `source/crime_stats/Crime-Statistics-2023-2024.xlsx` | Annual release |
| `source/crime_stats/Crime-Statistics-2024-202_Q1.xlsx` | Quarterly |
| `source/crime_stats/Crime-Statistics-2024-2025_Q2.xlsx` | Quarterly |
| `source/crime_stats/Crime-Statistics-2024_2025_Q3.xlsx` | Quarterly |
| `source/crime_stats/Crime-Statistics-2024-2025_Q4.xlsx` | Quarterly |

### What happens

**1. Multi-source merge with precedence rules**

Because SAPS releases data incrementally and corrects prior years in later publications, each fiscal year is assigned an authoritative source according to these precedence rules:

| Fiscal Years | Authoritative Source |
|---|---|
| 2008/09 – 2011/12 | annual_2021 (sole source) |
| 2012/13 | annual_2022 > annual_2021 |
| 2013/14 – 2022/23 | annual_2024 > annual_2022 > annual_2021 |
| 2023/24 – 2024/25 | quarterly_reconstructed (sole source) |

For 2023/24 and 2024/25, the four quarterly releases (Q1–Q4) are summed to reconstruct annual totals.

**2. Category name normalisation**

Crime category labels differ across releases (e.g. `"assault with the intent to inflict grievous bodily harm"` vs `"assault gbh"`). These are normalised to a fixed set of 44 canonical names using `category_map.csv`.

**3. Station name normalisation**

Station names are mapped to their district and province via `station_map.csv`, which covers all 1,196 known SAPS stations.

**4. Data quality audit**

An audit is run and results written to `saps_audit.md` and `saps_audit_flags.csv`. 20 flags were raised across two categories:

- **sparse_station** — stations with data in fewer than 3 of the 17 fiscal years (e.g. newly created stations such as Bhityi, Cacadu, Kariega).
- **yoy_outlier** — year-on-year changes exceeding ±350% for Murder, which typically reflect small-count volatility at low-crime rural stations.

### Output files

| File | Description |
|------|-------------|
| `saps_crimes_long.csv` | One row per (station, category, fiscal year) — 783,214 records |
| `saps_crimes_wide.csv` | Pivoted: one row per station with columns per category-year combination |
| `saps_combined_final.csv` | Merged/deduplicated final table before DB load |
| `saps_crimes.db` | SQLite database with two tables (see below) |
| `saps_audit.md` | Human-readable audit report |
| `saps_audit_flags.csv` | Machine-readable flag log |
| `category_map.csv` | raw name → canonical name lookup |
| `station_map.csv` | station name → district, province lookup |

**SQLite schema (`saps_crimes.db`):**

```sql
-- Normalised crime counts
crimes(station TEXT, crime_category TEXT, fiscal_year TEXT, count INTEGER)

-- Station metadata with GPS coordinates
stations(station TEXT, district TEXT, province TEXT, latitude REAL, longitude REAL)
```

Coverage: 1,196 stations × 44 crime categories × 17 fiscal years (2008/09 – 2024/25), totalling 783,214 records.

---

## Stage 2 — GIS Enrichment (`pipeline/enriched/`)

### Input files

| File | Description |
|------|-------------|
| `source/police_boundaries/boundaries/Police_bounds.shp` | Station precinct boundary polygons |
| `source/police_boundaries/points/Police_points.shp` | Station point locations |
| `pipeline/clean/saps_crimes.db` | Cleaned crime statistics |

### What happens

The crime statistics from the SQLite DB are pivoted into a wide format (`category_FYyyyy_yy` columns) and joined onto both shapefiles by station name. This produces two spatially-enriched datasets — one polygon layer (precinct boundaries) and one point layer (station locations) — each carrying the full 17-year crime history for all 44 categories as feature properties.

The result is re-projected and written out as both Shapefile and GeoJSON.

### Output files

| File | Description |
|------|-------------|
| `Police_bounds_enriched.geojson` / `.shp` | 1,167 precinct boundary polygons with all crime data attached |
| `Police_points_enriched.geojson` / `.shp` | 1,167 station point locations with all crime data attached |
| `SAPS_stations_combined.csv` | Tabular join of station metadata + crime totals |
| `SAPS_data_coverage.csv` | Per-fiscal-year record counts by source (annual vs quarterly) |

Each enriched feature contains the original shapefile attributes (`COMPNT_NM`, `STATION`, `CREATE_DT`, `VERSION`) plus one column per category-year combination, e.g. `Murder_FY2023_24`, `Burglary at Residential Premises_FY2019_20`, etc.

---

## Stage 3 — Build Output JSONs (`data/`)

The enriched data and SQLite DB are read by `build_report_data.py` (run locally, not committed) to produce the two static JSON files served by the web app.

### `data/stations.json`

An array of 1,167 station objects. Each station is a self-contained intelligence record pre-computed at build time, so the front-end needs no server-side queries.

**Fields per station:**

| Field | Description |
|-------|-------------|
| `id` | URL-safe slug (e.g. `"aberdeen"`) |
| `name`, `name_upper` | Display name and uppercase variant |
| `province`, `district` | Administrative location |
| `lat`, `lon` | GPS coordinates (from `Police_points` shapefile) |
| `risk_score` | Composite 0–100 score |
| `risk_label` | Label: LOW / MODERATE / HIGH RISK / VERY HIGH RISK |
| `primary_year` | Most recent fiscal year with data |
| `total` | Total incidents in `primary_year` |
| `total_yoy` | Year-on-year change vs prior year (decimal fraction) |
| `nat_rank` / `nat_total` | Station's national rank and total stations ranked |
| `prov_rank` / `prov_total` | Station's provincial rank and total stations in province |
| `nat_avg` / `prov_avg` | National and provincial average total incidents |
| `trend` | Array of 5 annual totals (most recent 5 fiscal years) |
| `nat_trend` / `prov_trend` | Corresponding national and provincial averages |
| `trend_years` | Fiscal year labels for the trend arrays |
| `trend_5yr_pct` | 5-year percentage change |
| `peak_year` / `peak_total` | Fiscal year and count of highest recorded total |
| `categories` | Array of per-category breakdowns (see below) |
| `total_incidents` | Alias for `total` |

**Per-category object (inside `categories` array):**

| Field | Description |
|-------|-------------|
| `name` | Canonical crime category name |
| `count` | Incidents in primary year |
| `yoy` | Year-on-year change (decimal fraction) |
| `nat_rank` / `nat_total` | National rank for this category |
| `nat_avg` / `prov_avg` | National and provincial averages for this category |
| `multiplier` | Station count as a multiple of national average (e.g. `2.1` = 2.1× the national avg) |
| `trend` | 5-year trend array for this category |

### `data/precincts.json`

*(Removed — not used by any live page.)*

This file was a curated dataset of 40 named precincts (neighbourhood-level groupings of stations). It contained supplementary fields — population estimates, area, response time, monthly breakdowns — intended for a precinct-level view that was never built into the app.

---

## Data Flow Summary

```
SAPS .xlsx files (7 releases)
        │
        ▼
   [Merge + precedence rules]
        │
   [Category normalisation via category_map.csv]
        │
   [Station normalisation via station_map.csv]
        │
   [Audit & flag outliers]
        │
        ▼
saps_crimes.db  ←────────────────────────────────────────────────┐
crimes(station, category, fiscal_year, count)  783,214 rows       │
stations(station, district, province, lat, lon)  1,196 rows       │
        │                                                          │
        │            Police_bounds.shp ──┐                        │
        │            Police_points.shp ──┴──► [Spatial join] ─────┘
        │                                          │
        │                                          ▼
        │                              Police_bounds_enriched.*
        │                              Police_points_enriched.*
        │                              SAPS_stations_combined.csv
        │
        ▼
   [build_report_data.py]
        │
        ├──► data/stations.json   (1,167 stations, all computed metrics)
        └──► data/precincts.json  (40 precincts, semi-curated)
```

---

## Key Numbers

| Item | Value |
|------|-------|
| Fiscal years covered | 17 (2008/09 – 2024/25) |
| SAPS stations | 1,196 (1,167 with shapefile match) |
| Crime categories | 44 canonical |
| Total crime records | 783,214 |
| Source files | 7 SAPS .xlsx releases |
| Audit flags | 20 (10 sparse stations, 10 YoY outliers) |
| Output stations | 1,167 in `stations.json` |
| Output precincts | 40 in `precincts.json` |
