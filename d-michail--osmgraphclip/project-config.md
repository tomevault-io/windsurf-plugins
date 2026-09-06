---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository. For a project overview, pretrained models, and training dataset information see `README.md`.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository. For a project overview, pretrained models, and training dataset information see `README.md`.

## Commands

### Setup
```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Build a dataset

Dataset creation is a two-step process:

1. **`create_dataset.py`** — downloads OSM GeoJSON files and populates the `downloads` table in `dataset.db`
2. **`create_graphs.py`** — builds graph pickles from the GeoJSON files and populates the `graphs` table in `dataset.db`

Both steps are independently resumable.

#### `create_dataset.py` — batch download

```bash
# From a CSV of locations
python create_dataset.py \
  --output-dir location_dataset \
  --locations-file data/locations.csv \
  --bbox-size 250

# Built-in city grid
python create_dataset.py --output-dir athens --city athens

# Adaptive bbox with profile
python create_dataset.py --output-dir dataset --locations-file data/locations.csv \
  --adaptive-bbox --bbox-profile rural

# Adaptive bbox with manual size control
python create_dataset.py --output-dir athens --city athens \
  --adaptive-bbox --bbox-size 250 --max-bbox-size 1000 \
  --bbox-expansion-strategy multiplicative --bbox-expansion-factor 1.5 \
  --min-total-nodes 5 --min-unique-labels 3 --min-richness-score 0.2

# Force full re-download
python create_dataset.py ... --no-resume
```

| Flag | Default | Description |
|---|---|---|
| `--output-dir` | required | Directory to store the dataset |
| `--locations-file` | — | CSV with `lat`/`lon` columns (mutually exclusive with `--city`) |
| `--city` | — | Built-in city name to generate a grid dataset |
| `--sample-spacing` | `500` | Grid spacing in metres (only with `--city`) |
| `--bbox-size` | `250` | Initial bbox half-width in metres; overrides `--bbox-profile` |
| `--bbox-profile` | — | Preset: `dense_city`, `suburb`, `rural`, `wilderness`, `regional` |
| `--tagw-path` | `data/all_tags30_frequency1.json` | Tag weights JSON (needed for richness scoring with `--adaptive-bbox`) |
| `--adaptive-bbox` | off | Expand bbox until richness thresholds are met |
| `--max-bbox-size` | — | Absolute bbox ceiling in metres (overrides `--adaptive-max-factor`) |
| `--adaptive-max-factor` | `4.0` | Ceiling = initial × factor (used when `--max-bbox-size` is not set) |
| `--bbox-expansion-strategy` | `multiplicative` | `multiplicative` or `linear` |
| `--bbox-expansion-factor` | `1.5` | Growth factor per retry (multiplicative strategy) |
| `--bbox-expansion-step` | `100` | Step in metres per retry (linear strategy) |
| `--min-total-nodes` | `5` | Minimum OSM feature count |
| `--min-unique-labels` | `3` | Minimum distinct OSM tag labels |
| `--min-richness-score` | `0.2` | Minimum composite richness score [0, 1] |
| `--multiresolution` | off | Download at multiple bbox levels per location |
| `--levels` | `200 500 2000 5000 20000` | Bbox sizes in metres for each level (fine → coarse) |
| `--best-level-only` | off | Keep only the richest level per location (requires `--multiresolution`) |
| `--best-level-richness-weight` | `1.0` | Weight for composite richness in best-level scoring |
| `--best-level-depth-weight` | `0.2` | Weight for semantic depth (avg non-null tags per feature) |
| `--best-level-categories-weight` | `0.1` | Weight for category coverage |
| `--best-level-size-weight` | `0.1` | Penalty for bbox size (prefers finer levels) |
| `--best-level-node-weight` | `0.5` | Penalty for node count (targets ~20–30 nodes) |
| `--best-level-min-nodes` | `10` | Levels below this node count receive a large penalty |
| `--save-road-graph` | off | Also save OSM road graph as GraphML |
| `--backend` | `overpass` | Data source: `overpass`, `postgis`, or `auto` (PostGIS → Overpass fallback) |
| `--postgis-url` | — | PostgreSQL DSN for PostGIS backend (or `POSTGIS_URL` env var) |
| `--postgis-max-rows` | `50000` | Max rows per geometry table from PostGIS; 0 = unlimited |
| `--nodata-retry-after` | — | Retry `.nodata` sentinel files older than N hours |
| `--resume` / `--no-resume` | resume | Skip already downloaded locations |
| `--workers` | `4` | Parallel worker threads (keep low for public Overpass API) |
| `--debug` | off | Enable DEBUG-level logging |

#### `create_graphs.py` — build graph pickles

```bash
# Default (CLIP embeddings)
python create_graphs.py \
  --output-dir location_dataset \
  --tagw-path data/all_tags30_frequency1.json

# SBERT backend — update node_embedding_dim in configs/default.yaml to match
python create_graphs.py --output-dir location_dataset \
  --tagw-path data/all_tags30_frequency1.json --embedding-backend sbert

# Force full rebuild
python create_graphs.py --output-dir location_dataset \
  --tagw-path data/all_tags30_frequency1.json --no-resume
```

| Flag | Default | Description |
|---|---|---|
| `--output-dir` | required | Same directory used with `create_dataset.py` |
| `--tagw-path` | `data/all_tags30_frequency1.json` | Tag weights JSON for graph construction |
| `--embedding-backend` | `clip` | `clip` (512-dim) or `sbert` (384/768-dim) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [d-michail/osmgraphclip](https://github.com/d-michail/osmgraphclip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
