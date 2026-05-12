# LA County Wildfire Risk Map

An interactive map comparing per-building wildfire risk scores across LA County using multiple fire behavior modeling methodologies.

**Live map:** https://meimeicollins2000.github.io/open-wildfire-risk/wildfire-risk.html

---

## What it shows

Each dot represents a sampled building centroid (~100,000 buildings) colored by **Risk to Potential Structures (RPS)** — the annualized expected loss from wildfire. A dropdown lets you switch between risk metrics derived from three independent burn probability datasets, revealing how methodology choice shifts community-level risk estimates.

Fields starting with `rps_` use a white-to-red color scale. All other fields use a green-to-red scale. Click any dot to see all field values for that building.

---

## Data sources

| Field(s) | Source |
|:---|:---|
| `rps_scott`, `crps_scott` | [Scott et al. 2024](https://www.fs.usda.gov/rds/archive/catalog/RDS-2020-0016-2) — USDA Forest Service / Pyrologix |
| `rps_riley`, `bp_*_riley` | [Riley et al. 2025](https://www.fs.usda.gov/rds/archive/catalog/RDS-2025-0006) — USDA Forest Service |
| `usda_bp` | USDA burn probability COG, processed from Scott et al. 2024 and posted to [Source Cooperative](https://source.coop/espm-288/espm-288-testing) by Mei Collins for ESPM-288 |
| `rps_usda` | Composite: `usda_bp × crps_scott` (this analysis) |
| Building footprints | [Overture Maps Foundation](https://overturemaps.org/) |
| Risk scores joined to buildings | [CarbonPlan Open California Risk (OCR)](https://docs.carbonplan.org/ocr/en/latest/reference/data-sources.html) |

---

## How to reproduce

### Requirements

- R 4.5+
- Quarto
- R packages: `duckdb`, `DBI`, `sf`, `terra`, `dplyr`, `jsonlite`, `mapgl`, `htmlwidgets`

### Credentials

The `usda_bp` raster requires Source Cooperative temporary credentials. Obtain them from the [ESPM-288 collection page](https://source.coop/espm-288/espm-288-testing) and save to `credentials.json` at the repo root (this file is gitignored):

```json
{
  "credentials": {
    "aws_access_key_id": "...",
    "aws_secret_access_key": "...",
    "aws_session_token": "...",
    "region_name": "us-west-2"
  }
}
```

### Render

```bash
quarto render wildfire-risk.qmd
```

This downloads all 13 LA county GeoParquet files from CarbonPlan, samples 100,000 building centroids, extracts the USDA burn probability raster value at each point, and produces a self-contained `wildfire-risk.html`.

---

## Deployment

The repo is deployed to GitHub Pages via GitHub Actions (`.github/workflows/static.yml`). Any push to `main` triggers a redeploy.

---

*Coding assistance provided by [Claude](https://claude.ai) (Anthropic).*
