# India Crop Yield Explorer

**Live tool:** https://px-d-assignment-jdj85lxqt-likith-narakala-s-projects.vercel.app/

A full-screen, swipeable district map of India for comparing crop yield, sown area, and IMD monsoon rainfall side by side. Built as a static site (`index.html`) on top of a small data-cleaning pipeline of three notebooks.

## What it does

- Colour any district by **yield (t/ha)**, **sown area (ha)**, **IMD rainfall (mm)**, or **rainfall anomaly (%)**.
- Set an independent **crop / season / year** scenario on the left and right, then drag the slider to wipe one map over the other.
- Hover any district for both scenario values, the difference, and — if a crop statistic is missing — a plain-English reason why it's grey instead of a guess.
- An in-app **Methodology** page documents exactly how each dataset is cleaned, joined, and how every edge case (garbled district names, new district splits, incomplete rainfall seasons, etc.) is handled.

## Data sources

| Dataset | Source | Cleaned by |
|---|---|---|
| Crop statistics | DES crop-wise area/production/yield yearbook, 1997–98 to 2022–23 | `crop_clean.ipynb` → `data/processed/crop_clean.csv` |
| Rainfall | IMD 0.25° daily rainfall grids, 1997–2023 | `imd_clean.ipynb` → `data/processed/rainfall_district_seasonal.csv` |
| Boundaries | `data/INDIA_DISTRICTS.geojson` (current district polygons) | used to join both of the above |

`build_panel.ipynb` merges the two cleaned tables into the single file the app actually reads: `data/processed/district_panel.csv`.

No value is ever invented to fill a gap — a district with no yearbook row for a given crop/season/year stays grey, and the tooltip explains why.

## Project structure

```
index.html                  the whole front end (static, no build step)
crop_clean.ipynb            cleans the crop yearbook, matches district names onto the map
imd_clean.ipynb             downloads IMD rainfall grids, aggregates to district/season totals
build_panel.ipynb           merges the two into data/processed/district_panel.csv
Crop_data/                  raw crop yearbook CSV + codebook
data/                       district/state GeoJSON, IMD downloads, processed outputs
```

## Running the pipeline

Notebooks must run in this order (each writes a file the next one reads):

1. `crop_clean.ipynb`
2. `imd_clean.ipynb` (downloads IMD grids on first run — slow)
3. `build_panel.ipynb`

```bash
pip install -r requirements.txt
```

## Running the site locally

The front end is static but uses `fetch()`, so it needs to be served over HTTP, not opened as a `file://` URL:

```bash
python -m http.server 8080
```

Then open `http://127.0.0.1:8080`.

## Deployment

Deployed on Vercel as a static site — no framework, no build command. `vercel.json` pins the project to static hosting so Vercel doesn't try to auto-detect it as a Python app (the pipeline's `requirements.txt` otherwise confuses that detection). `.vercelignore` keeps the notebooks and raw/intermediate data files out of the deployed bundle; only `index.html`, the two GeoJSON files, and `data/processed/district_panel.csv` are served.

## Responsible use of Gen AI

Generative AI tools were used as coding assistants during development, with all logic, data-cleaning decisions, and edge-case handling reviewed and directed by me:

- **Claude Code** and **Cursor** — used for writing and editing code (the notebooks and the front end).
- **Vercel** — used to deploy the static site.
