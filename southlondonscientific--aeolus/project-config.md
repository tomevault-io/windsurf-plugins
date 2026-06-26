---
trigger: always_on
description: Air quality data downloading and standardisation library for UK and international monitoring networks.
---

# Aeolus - Claude Code Context

Air quality data downloading and standardisation library for UK and international monitoring networks.

**Current Version:** 0.4.5.4

## Quick Start

```bash
# Activate virtual environment
source .venv/bin/activate

# Run tests
pytest

# Run a specific test file
pytest tests/test_airqo.py -v
```

## Environment Setup

**Always activate the virtual environment before working:**
```bash
source .venv/bin/activate
```

**Environment variables** are in `.env` (copy from `.env.example`):
- `OPENAQ_API_KEY` - Required for OpenAQ data
- `BL_API_KEY` - Required for Breathe London data
- `AIRQO_API_KEY` - Required for AirQo data
- `PURPLEAIR_API_KEY` - Required for PurpleAir data
- `AIRNOW_API_KEY` - Required for EPA AirNow data
- AURN, SAQN, and Sensor.Community do not require API keys

## Project Structure

```
notebooks/                   # 8 user story Jupyter notebooks (v0.4.0)
src/aeolus/
├── __init__.py          # Public API (download, list_sources, etc.)
├── api.py               # Main download() function implementation
├── registry.py          # Source registration system
├── transforms.py        # Data normalisation utilities
├── sources/             # Data source implementations
│   ├── regulatory.py    # UK regulatory networks (AURN, SAQN, WAQN, NI, AQE)
│   ├── laqn.py          # London Air Quality Network (ERG/Imperial)
│   ├── openaq.py        # OpenAQ global portal
│   ├── breathe_london.py # Breathe London network
│   ├── airqo.py         # AirQo African network
│   ├── purpleair.py     # PurpleAir global portal
│   ├── sensor_community.py # Sensor.Community citizen science
│   ├── airnow.py        # EPA AirNow US network
│   ├── eea.py           # EEA European monitoring network
│   ├── sonitus.py       # Smart Dublin (Sonitus) network
│   ├── sos.py           # UK-AIR SOS near-real-time API (AURN-SOS, SAQN-SOS, etc.)
│   └── _sos_mapping.json # Static SOS station mapping (auto-generated)
├── cache.py             # Local Parquet-based download caching
├── geo.py               # Geospatial utilities (haversine, bbox)
├── progress.py          # Optional tqdm progress bars (fallback to logging)
├── metrics/             # Air quality metrics calculations
└── viz/                 # Visualisation utilities
```

## Data Sources

### Networks (known site lists)

| Source | API Key | Coverage |
|--------|---------|----------|
| AURN | No | UK national network |
| SAQN | No | Scotland |
| WAQN | No | Wales |
| NI | No | Northern Ireland |
| AQE | No | Air Quality England |
| LAQN | No | London Air Quality Network (~250 sites) |
| BREATHE_LONDON | Yes (`BL_API_KEY`) | London low-cost sensors |
| AIRQO | Yes (`AIRQO_API_KEY`) | African cities (200+ sensors) |
| AIRNOW | Yes (`AIRNOW_API_KEY`) | USA, Canada, Mexico |
| SENSOR_COMMUNITY | No | Global citizen science (35,000+) |
| EEA | No | Europe (40+ countries, 7,000+ stations) |
| SONITUS | No | Dublin, Ireland |

### Portals (search required)

| Source | API Key | Coverage |
|--------|---------|----------|
| OPENAQ | Yes (`OPENAQ_API_KEY`) | Global (100+ countries) |
| PURPLEAIR | Yes (`PURPLEAIR_API_KEY`) | Global low-cost sensors (30,000+) |

### Optional SDK Dependencies

OpenAQ and PurpleAir require optional SDK packages not available on conda-forge:

| Extra | Install command | Provides |
|-------|----------------|----------|
| `openaq` | `pip install aeolus_aq[openaq]` | OpenAQ portal access |
| `purpleair` | `pip install aeolus_aq[purpleair]` | PurpleAir portal access |
| `stats` | `pip install aeolus_aq[stats]` | `statsmodels` for deseasonalisation in `trend()` |
| `progress` | `pip install aeolus_aq[progress]` | `tqdm` progress bars for bulk downloads |
| `all` | `pip install aeolus_aq[all]` | OpenAQ + PurpleAir + statsmodels |

For conda users: `conda install -c conda-forge aeolus_aq` then `pip install openaq purpleair-api` for portal sources.

## Standard Data Schema

All sources normalise data to this 8-column schema:
- `site_code` - Unique site identifier
- `date_time` - Timestamp (UTC-aware, left-closed intervals)
- `measurand` - Pollutant (PM2.5, NO2, O3, etc.)
- `value` - Measurement value
- `units` - Units (typically ug/m3)
- `source_network` - Data source name
- `ratification` - Data quality flag
- `created_at` - When record was fetched (UTC-aware)

**Metadata schema** (from `get_metadata()` / `find_sites()`):
- `site_code` - Unique site identifier (use for download)
- `site_name` - Human-readable name
- `latitude`, `longitude` - Location coordinates
- `source_network` - Data source name

**Bounding box format** (consistent across all sources):
- `bbox=(min_lon, min_lat, max_lon, max_lat)` - GeoJSON/shapely convention

## Common Commands

```bash
# Install from conda-forge
conda install -c conda-forge aeolus_aq

# Install with all optional sources (pip only)
pip install aeolus_aq[all]

# Install in development mode
pip install -e ".[dev]"

# Run all tests
pytest

# Run tests with coverage
pytest --cov=aeolus --cov-report=html

# Run specific test markers
pytest -m "not slow"        # Skip slow tests
pytest -m "not integration" # Skip API-dependent tests

# Run demos
python demo.py              # Main demo
python demo_airqo.py        # AirQo demo

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [southlondonscientific/aeolus](https://github.com/southlondonscientific/aeolus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
