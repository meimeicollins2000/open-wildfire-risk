# Map Rendering Guide: Interactive Choropleth via GitHub Pages

## How it works (end to end)

1. An R Quarto document (`.qmd`) creates a map widget using the `mapgl` package.
2. Quarto renders it to a **self-contained HTML file** (all JS/CSS/data inline — no server needed).
3. That HTML file is committed to the repo.
4. A GitHub Actions workflow deploys the entire repo as a static site to GitHub Pages whenever you push to `main`.

---

## What the map needs

### Data format

Your data must be an **`sf` polygon object** in R — i.e., a data frame where each row is a geographic polygon with attributes. `sf` reads from GeoJSON, Shapefile, GeoParquet, FlatGeobuf, etc.

```r
library(sf)
my_polygons <- read_sf("data/my_data.geojson")  # or any supported format
```

Each polygon should have at least one numeric or categorical column you want to visualize as fill color.

---

### R packages required

```r
library(mapgl)   # MapLibre GL JS wrapper — the map renderer
library(sf)      # Polygon data handling
```

`mapgl` renders via **MapLibre GL JS** (WebGL). No API key needed when using `maplibre()` with a Carto basemap style.

---

### Quarto document setup

The YAML front matter must include `embed-resources: true` so the output HTML is fully self-contained:

```yaml
---
title: "My Map"
format:
  html:
    embed-resources: true
---
```

---

## Map code patterns

### Categorical fill (e.g., letter grades, categories)

```r
maplibre(
  style  = carto_style("positron"),   # free basemap, no API key
  center = c(longitude, latitude),    # [lon, lat] of map center
  zoom   = 12
) |>
  add_fill_layer(
    id     = "my-layer",
    source = my_polygons,             # sf object
    fill_color = match_expr(
      column = "category_column",
      values = list("A",       "B",       "C"),
      stops  = list("#76a865", "#7cb5bd", "#d9838d")
    ),
    fill_opacity       = 0.65,
    fill_outline_color = "white"
  ) |>
  add_tooltips("my-layer", "category_column") |>
  add_legend(
    "Legend Title",
    values = c("A", "B", "C"),
    colors = c("#76a865", "#7cb5bd", "#d9838d"),
    type   = "categorical"
  )
```

### Continuous fill (e.g., a numeric score or index)

```r
maplibre(style = carto_style("positron"), center = c(lon, lat), zoom = 12) |>
  add_fill_layer(
    id     = "my-layer",
    source = my_polygons,
    fill_color = interpolate(
      column = "numeric_column",
      values = c(0, 0.5, 1),           # breakpoints in data space
      stops  = c("#d73027", "#ffffbf", "#1a9850")  # colors at those breakpoints
    ),
    fill_opacity = 0.8
  ) |>
  add_legend("Legend Title", values = c(0, 1), colors = c("#d73027", "#1a9850"))
```

### 3D extrusion (height encodes a value)

```r
maplibre(
  style = carto_style("dark-matter"),
  center = c(lon, lat),
  zoom = 12,
  pitch = 50                            # camera tilt in degrees
) |>
  add_fill_extrusion_layer(
    id     = "my-layer",
    source = my_polygons,
    fill_extrusion_color = interpolate(
      column = "numeric_column",
      values = c(0, 1),
      stops  = c("#d73027", "#1a9850")
    ),
    fill_extrusion_height = interpolate(
      column = "numeric_column",
      values = c(0, 1),
      stops  = c(0, 800)               # height in meters
    ),
    fill_extrusion_opacity = 0.85
  )
```

---

## Rendering to HTML

Run in R (or render via terminal):

```r
quarto::quarto_render("my_map.qmd")
# → produces my_map.html (self-contained, ~3–10 MB)
```

Or from the terminal:
```bash
quarto render my_map.qmd
```

Commit the resulting `.html` file to the repo root.

---

## GitHub Pages deployment

Add this workflow file at `.github/workflows/static.yml`:

```yaml
name: Deploy static content to Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
      - id: deployment
        uses: actions/deploy-pages@v4
```

Then enable GitHub Pages in the repo settings:
**Settings → Pages → Source → GitHub Actions**

The map will be live at `https://<username>.github.io/<repo-name>/my_map.html`.

---

## Summary checklist

- [ ] `sf` polygon object with at least one attribute column to visualize
- [ ] Quarto `.qmd` with `embed-resources: true` in YAML
- [ ] Map built with `mapgl::maplibre()` + `add_fill_layer()` (or extrusion variant)
- [ ] Rendered `.html` file committed to repo root
- [ ] `.github/workflows/static.yml` present
- [ ] GitHub Pages enabled (Settings → Pages → GitHub Actions)
