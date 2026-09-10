---
trigger: always_on
description: PowerGenome is a **capacity expansion modeling data pipeline** that generates input files for power system optimization models (primarily GenX). It transforms raw energy data (EIA, NREL ATB, PUDL) into model-ready datasets for studying electricity system futures across different scenarios, regions, and planning years.
---

# PowerGenome AI Coding Agent Instructions

## Project Overview

PowerGenome is a **capacity expansion modeling data pipeline** that generates input files for power system optimization models (primarily GenX). It transforms raw energy data (EIA, NREL ATB, PUDL) into model-ready datasets for studying electricity system futures across different scenarios, regions, and planning years.

**Key insight**: This is NOT a simulator—it's a sophisticated ETL pipeline that clusters generators, builds transmission constraints, creates demand profiles, and manages multi-scenario configurations through YAML-driven workflows.

## Architecture & Data Flow

### Core Pipeline Pattern

1. **Settings → DataManager → Module-specific Processing → GenX Output Files**
2. All data flows through the **DataManager singleton** (in-memory DuckDB) that provides standardized table access
3. Entry point: `run_powergenome` CLI → `run_powergenome.py:main()`

### Major Components

```
powergenome/
├── database.py          # DataManager singleton - centralizes ALL data access
├── settings.py          # Settings class handles YAML configs + scenario management
├── generators.py        # GeneratorClusters class - clusters existing plants, adds new resources
├── GenX.py             # Formats data for GenX model (resource tags, time reduction)
├── load_profiles.py    # Demand curves, DG subtraction, growth projections
├── transmission.py     # Inter-regional constraints, line loss, expansion costs
├── fuels.py            # Fuel price time series from EIA AEO API
├── new_build.py        # New-build resource costs and configuration
├── resource_clusters.py # ClusterBuilder for renewable resource site selection
└── distributed_gen.py   # Rooftop solar capacity & profiles (recent refactor)
```

### Data Architecture (Critical!)

**Only use the Modern DataManager**:

1. **Legacy**: External SQLite databases referenced via `.env` file (`PUDL_DB`, `PG_DB`)
2. **Modern**: DataManager loads from folder/DB configured in settings YAML

**Table Configuration** in settings supports:
- Simple: `generation_table: "generators.csv"`
- Advanced: Dictionary with `table_name`, `columns`, `filters` (DNF logic), `scenario` (convenience filter)

Example:
```yaml
demand_table:
  table_name: demand_timeseries.parquet
  scenario: HighEV
  filters:
    - - [region, '=', 'CA_N']
      - [year, '>=', 2030]
```

The DataManager normalizes these to standardized names (`generation`, `demand`, etc.) accessible via `get_data()`.

## Critical Conventions

### Settings Management

- **Multi-scenario workflows** use `scenario_definitions_fn` CSV to define case variations
- `settings_management` in YAML dynamically swaps parameter values per scenario
- Settings are **frozen dictionaries** after loading—use `Settings` class methods to update
- Model regions defined via `model_regions` + `region_aggregations` (aggregates IPM regions)
- Planning years: `model_year` + `model_first_planning_year` define period ranges

### Generator Clustering

**Existing generators**:
- Clustered by region + technology using k-means (heat rate, capacity as features)
- `num_clusters` sets default, `alt_num_clusters` overrides per region/tech
- `tech_groups` merges similar technologies (e.g., landfill gas + municipal waste)
- `retirement_ages` filters by plant age calculated from operating year
- **Critical**: In myopic models, set retirement ages high (500+) to avoid cluster membership changes between periods

**New generators**:
- Identified by `<technology>_<tech_detail>_<cost_case>` strings
- `new_resources` lists available technologies with unit sizes
- `resource_modifiers` adjusts costs/parameters in-place
- `modified_new_resources` creates renamed copies with modified params
- Renewables use `renewable_clusters` to define resource sites via pre-computed wind/solar profiles

### Regional Patterns

- **Base regions** are the base geography (from a source like ReEDS or IPM)
- **Model regions** aggregate base regions via `region_aggregations`
- Many settings use model region names as keys: `regional_tag_values`, `new_gen_not_available`
- String matching is case-insensitive for technology names but **exact** for regions

### Data Paths

Put these in a settings YAML file (e.g., `env.yml`):
```yaml
data_location: /path/to/data_folder  # Used by DataManager
RESOURCE_GROUP_PROFILES: /path/to/generation_profiles
DISTRIBUTED_GEN_DATA: /path/to/dg_profiles  # Legacy, prefer settings tables
RESOURCE_GROUPS: /path/to/resource_groups   # Can override in settings YAML
```

## Development Workflows

### Running Tests

```bash
# From repo root, activate environment first
conda activate powergenome
pytest tests/                          # All tests
pytest tests/generators_test.py -v    # Specific module
pytest -k "test_cluster" -v           # Pattern matching
```

Tests use **fixtures** with temporary data files. DataManager must be initialized in test setup:
```python
from powergenome.database import initialize_data_manager

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PowerGenome/PowerGenome](https://github.com/PowerGenome/PowerGenome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
