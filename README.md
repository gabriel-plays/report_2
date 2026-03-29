# SafeZone Intelligence

Property crime risk intelligence for South African addresses, powered by official SAPS crime statistics across 1,196 stations and 17 fiscal years (2008/09 – 2024/25).

## Live site

Deploy to GitHub Pages — 100% static, no server required.

## How it works

1. **`index.html`** — Address search page. Enter any South African address to find the nearest SAPS station and view its risk profile.
2. **`report.html`** — Station intelligence report. Full narrative-led analysis including risk ranking, 5-year trend, category breakdown, and recommendations. Accessed via `report.html?id=<station-slug>`.
3. **`data/stations.json`** — Pre-computed intelligence for all 1,196 stations: national rank, provincial rank, risk score, 5-year trends, per-category multipliers vs national average, YoY changes, GPS coordinates.

## Project structure

```
index.html              ← Search UI & station finder
report.html             ← Station crime intelligence report
data/
  stations.json         ← All station data (pre-computed, ~1.2 MB)
  precincts.json        ← Station boundary polygons
.nojekyll               ← Prevents GitHub Pages Jekyll processing
```

## Deploy to GitHub Pages

1. Push this folder to a GitHub repository
2. Go to **Settings → Pages**
3. Source: **Deploy from a branch** → `main` → `/ (root)`
4. Your site will be live at `https://<username>.github.io/<repo>/`

## Data

All statistics are sourced from official SAPS Annual Crime Statistics releases (2020/21 – 2024/25) and quarterly reports, processed through a normalisation pipeline. Coverage: 1,196 stations, 44 crime categories, fiscal years 2008/09 to 2024/25.

The `data/stations.json` file is pre-built from the SQLite database (`clean_data/saps_crimes.db`, not included in this repo). To rebuild it, run `build_report_data.py` locally with the database in place.

## Tech stack

- Vanilla HTML / CSS / JS — zero build step, zero dependencies
- Chart.js (CDN) — bar, donut, and line charts
- Google Fonts (CDN) — Inter, IBM Plex Mono, Libre Baskerville
