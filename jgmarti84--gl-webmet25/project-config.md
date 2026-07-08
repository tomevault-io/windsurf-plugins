---
trigger: always_on
description: > **CRITICAL:** You are operating in a multi-root workspace alongside the `radarlib` repository.
---

# webmet25 — Copilot Instructions

## 🧠 Context & Knowledge Base
> **CRITICAL:** You are operating in a multi-root workspace alongside the `radarlib` repository. 
> Before writing code, route your knowledge by reading the appropriate documentation:

- `docs/DISCOVERY_REPORT.md`: Full technical analysis, DB schema, API endpoints, and identified risks.
- `README.md` (root): High-level purpose, tech stack, and architecture overview.
- `docs/DATA_FLOW.md`: How `webmet25` ingests, processes, and displays `radarlib` data. **Read this before changing the indexer or API.**
- `docs/COMPONENTS.md`: Breakdown of UI/frontend modules. **Read this before making frontend/Leaflet changes.**

---

## About This Repository
webmet25 is the **data consumer** in the radarmet system. It is one 
of two repositories. It ingests Cloud-Optimized GeoTIFF (COG) files 
produced by **radarlib**, indexes them into a PostgreSQL/PostGIS 
database, and serves them via a REST API and interactive Leaflet 
frontend.

---

## System Context
```text
radarlib (producer)
    │
    ├── outputs GeoTIFF COGs to ROOT_RADAR_PRODUCTS_PATH
    ├── outputs GeoJSON tops & cores to TOPS_AND_CORES_DIR
    │
    ▼
webmet25 (consumer)
    │
    ├── Indexer watches ROOT_RADAR_PRODUCTS_PATH
    ├── TopsAndCoresWatcher watches TOPS_AND_CORES_DIR
    │   ├── parses filenames
    │   ├── extracts metadata
    │   └── stores in PostgreSQL/PostGIS
    │
    ├── FastAPI serves metadata + tiles
    │
    └── Leaflet frontend renders radar map
```

### radarlib Output Contract (what we depend on)
> ⚠️ webmet25 depends entirely on radarlib's output format.
> Never assume a different format without checking radarlib's 
> `docs/radarlib_EN.md` Output Contract section first.
> ⚠️ This contract is sourced from radarlib. If radarlib 
> changes its output format, update this section immediately.
> In case we are working on a multi-root workspace, you can directly inspect `radarlib` code to verify this.

### Primary Output Format
- **GeoTIFF (COG):** This is the primary and current output format.
  Cloud-Optimized GeoTIFF is the production standard.
- **PNG:** Deprecated. Kept only for backward compatibility.
  Do not build new features around PNG output.

### File Naming Convention

#### Current Production Format (Pattern 0)
`<RADAR_NAME>_<STRATEGY>_<VOL_NR>_<TIMESTAMP>_<FIELD>[o].<ext>`

| Token | Description | Example |
|-------|-------------|---------|
| `RADAR_NAME` | Radar station identifier | `RMA1` |
| `STRATEGY` | 4-digit scanning strategy code | `0315` |
| `VOL_NR` | 2-digit volume number | `01`, `02`, `04` |
| `TIMESTAMP` | ISO 8601 format: `YYYYMMDDTHHMMSSZ` | `20260401T205000Z` |
| `FIELD` | Radar field/variable name | `ZDR`, `DBZH` |
| `[o]` | Letter `o` suffix = raw/non-filtered data. Absent = filtered data | `ZDRo` vs `ZDR` |
| `ext` | File extension | `tif` (primary), `png` (deprecated) |

Examples:
```
RMA1_0315_01_20260401T205000Z_DBZH.tif     # filtered reflectivity, vol 01
RMA1_0315_01_20260401T205000Z_DBZHo.tif    # unfiltered (raw) reflectivity, vol 01
RMA1_0315_04_20260401T205000Z_COLMAX.tif   # column max, vol 04 (vigilant)
```

#### Legacy Format (Pattern 1 — backward compatibility only)
`<RADAR_NAME>_<TIMESTAMP>_<FIELD>[o]_<ELEVATION>.<ext>`

Example: `RMA1_20260401T205000Z_ZDRo_00.tif`

Legacy files are indexed with `strategy=None` and `vol_nr=None`. The indexer
logs a WARNING for each legacy file encountered.

PNG equivalent (deprecated, backward compat only):
`RMA1_20260401T205000Z_ZDR_00.png`

### Folder Structure
```text
ROOT_RADAR_PRODUCTS_PATH/
└── <RADAR_NAME>/
    └── /YYYY/
        └── /MM/
            └── /DD/
                ├── RMA1_20260401T205000Z_ZDR_00.tif
                ├── RMA1_20260401T205000Z_ZDRo_00.tif
                └── RMA1_20260401T205000Z_ZDR_00.png ← deprecated
```

### GeoTIFF Metadata Fields

| Field | Value | Purpose |
|---|---|---|
| **CRS** | EPSG:3857 | Web Mercator (Pseudo-Mercator). Frames endpoint
|         |           | converts to WGS84 for X-Bbox-* headers. |
| **radarlib_cmap** | Colormap name string | Name of matplotlib colormap used (e.g., `"grc_th"`) |
| **vmin** | Float | Minimum data value for color scaling |
| **vmax** | Float | Maximum data value for color scaling |
| **field_name** | String | Radar field name (e.g., `"DBZH"`) |
| **timestamp** | ISO 8601 | Data acquisition timestamp |

### Critical Rules
- **Never change this contract without updating webmet25 indexer.**
- **Do not add new output formats without updating both repos.**
- When implementing multi-elevation support in the future, the
  `ELEVATION` token must remain zero-padded to 2 digits (e.g.,
  `05`, `10`) to preserve consistent file naming.
- PNG generation should not be extended or improved. If a task
  involves PNG output, flag it and ask for confirmation.

> ⚠️ webmet25 depends entirely on radarlib's output format.
> Never assume a different format without checking radarlib's 
> `docs/radarlib_EN.md` Output Contract section first.

- **Primary format:** Cloud-Optimized GeoTIFF (.tif)
- **PNG:** Deprecated, backward compatibility only
- **File naming:**

---

## Tech Stack
- **Language:** Python 3.11
- **Backend:** FastAPI, SQLAlchemy, Alembic, GeoAlchemy2, Uvicorn

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jgmarti84/gl-webmet25](https://github.com/jgmarti84/gl-webmet25) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
