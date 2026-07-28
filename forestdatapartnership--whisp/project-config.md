---
trigger: always_on
description: This file is intended for both AI assistants (such as GitHub Copilot) and human contributors.
---

# Whisp Copilot Instructions

**Note:**
This file is intended for both AI assistants (such as GitHub Copilot) and human contributors.
It defines project-specific coding standards, architectural guidelines, and best practices to ensure consistency and quality in all code contributions.

## Project Overview
Whisp ("What is in that plot?") is a Python package for forest monitoring and deforestation risk assessment using Google Earth Engine (GEE). It implements the "Convergence of Evidence" approach by analyzing multiple satellite datasets to assess plots for compliance with deforestation-related regulations like EUDR.

**Key capabilities**: Process GeoJSON geometries through GEE to extract zonal statistics from 50+ datasets covering tree cover, commodity plantations, and forest disturbances (before/after 2020), then apply risk algorithms for different commodities (coffee, cocoa, rubber, palm oil, soy, livestock, timber).

### Deployment Ecosystem
**CRITICAL**: This package powers multiple production systems. Breaking changes impact:
- **Whisp API** (https://whisp.openforis.org/): Main service endpoint using this package
- **QGIS plugin**: Desktop GIS integration consuming API
- **Dashboards**: Monitoring interfaces relying on API outputs
- **Whisp Map**: EarthMap-based visualization platform (https://whisp.earthmap.org/)

**Two user groups**:
1. **Direct users**: Run package in notebooks with own GEE credentials (smaller group)
2. **API consumers**: Use Whisp API endpoints (larger group, multiple platforms)

**Backward compatibility is essential** - changes to column names, output structure, or risk assessment logic affect all downstream systems.

## Architecture

### Core Pipeline Flow
1. **Input**: GeoJSON → [`stats.py`](src/openforis_whisp/stats.py) converts to Earth Engine FeatureCollections
2. **Dataset Preparation**: [`datasets.py`](src/openforis_whisp/datasets.py) functions (suffix `_prep`) prepare GEE Image/ImageCollection objects
3. **Dataset Combination**: [`datasets.py::combine_datasets()`](src/openforis_whisp/datasets.py) orchestrates dataset filtering and merging
4. **Statistics Extraction**: [`stats.py::whisp_formatted_stats_geojson_to_df()`](src/openforis_whisp/stats.py) runs zonal stats via `reduceRegions()`
5. **Risk Assessment**: [`risk.py::whisp_risk()`](src/openforis_whisp/risk.py) applies decision tree logic to generate risk columns

### Configuration-Driven Design
- [`lookup_datasets.csv`](src/openforis_whisp/parameters/lookup_datasets.csv) defines ALL datasets, context columns, and metadata used in Whisp:
  - `corresponding_variable` column documents which function provides each dataset (for comprehension only, not used in code)
  - Controls which datasets feed into risk calculations: `use_for_risk_pcrop` (perennial crops) and `use_for_risk_acrop` (annual crops), which are currently held identical because the perennial/annual split is not yet wired (together they act as one combined crop-risk flag); plus `use_for_risk_timber` (timber)
  - Defines themes: `treecover`, `commodities`, `disturbance_before`, `disturbance_after`, `context_and_metadata`
  - National datasets use ISO2 codes in `ISO2_code` column; global datasets leave blank
- [`config_runtime.py`](src/openforis_whisp/parameters/config_runtime.py) defines output column names and formatting rules
- Schema validation via [`pd_schemas.py`](src/openforis_whisp/pd_schemas.py) using Pandera

## Critical Patterns

### Dataset Function Naming Convention
Functions in [`datasets.py`](src/openforis_whisp/datasets.py) **MUST** follow strict naming:
- Suffix: `_prep` (e.g., `g_jrc_gfc_2020_prep`)
- Prefix: `g_` for global datasets, `nXX_` for national (XX = ISO2 code, e.g., `nCI_bnetd_cocoa_prep` for Côte d'Ivoire)
- Return: `ee.Image` with `.rename('DatasetName')` matching CSV `name` column

**Example from CSV → Function mapping:**
```csv
name,corresponding_variable
EUFO_2020,g_jrc_gfc_2020_prep
Cocoa_bnetd,nCI_bnetd_cocoa_prep
```

### Earth Engine Best Practices
- **Avoid `.getInfo()`**: Keep operations server-side until final `convert_ee_to_df()` call
- **No loops over features**: Use `map()` and `reduceRegions()` for batch processing
- **Cache expensive images**: See `get_water_flag_image()` and `get_admin_boundaries_fc()` in [`stats.py`](src/openforis_whisp/stats.py) - reuses global datasets across all features instead of recreating per-feature
- **Date filtering**: Use module-level `CURRENT_YEAR` constant (from `datetime.now().year`) to avoid repeated calls

### Unit Handling System
Whisp supports both **hectares** and **percent** units:
- Unit type stored in column defined by `stats_unit_type_column` (default: `"Unit"`)
- [`risk.py::detect_unit_type()`](src/openforis_whisp/risk.py) auto-detects or accepts `explicit_unit_type` override
- All rows in a DataFrame **must** use same unit type (no mixing)
- Risk thresholds (e.g., `ind_1_pcent_threshold`) are percentage thresholds regardless of unit type

### Risk Assessment Logic
[`risk.py::whisp_risk()`](src/openforis_whisp/risk.py) implements commodity-specific decision trees:
- **Perennial crops** (coffee, cocoa, rubber, palm): Uses `Risk_PCrop` output
- **Annual crops** (soy): Uses `Risk_ACrop`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [forestdatapartnership/whisp](https://github.com/forestdatapartnership/whisp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
