---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenICU is an open-source Python framework for extracting, preprocessing, and analyzing ICU time series data from diverse datasets (MIMIC, eICU, etc.). The project converts heterogeneous ICU data into the standardized MEDS (Medical Event Data Standard) format, enabling reproducible research workflows.

**Key Goals:**
- Support multiple ICU data sources (public datasets like MIMIC, eICU, and custom institutional data)
- Extract medical concepts using declarative YAML configurations
- Export to MEDS format for standardized analysis
- Operate fully offline for medical data privacy compliance

## Development Commands

### Environment Setup
```bash
# The project uses uv for dependency management
# Dependencies are already installed in the dev container

# Install dependencies manually if needed:
uv sync
uv sync --all-groups  # Install with dev and docs groups
```

### Testing
```bash
# Run all tests with coverage
uv run pytest

# Run tests without coverage reports
uv run pytest --no-cov

# Run a specific test file
uv run pytest tests/test_example.py

# Run a specific test
uv run pytest tests/test_example.py::test_function_name
```

### Code Quality
```bash
# Format code with ruff
uv run ruff format

# Lint code with ruff
uv run ruff check

# Lint with auto-fix
uv run ruff check --fix

# Type checking with mypy
uv run mypy src/
```

### Documentation
```bash
# Serve documentation locally
uv run mkdocs serve

# Build documentation
uv run mkdocs build
```

### Jupyter Notebooks
```bash
# Launch JupyterLab for examples
uv run jupyter lab

# Example notebooks are in example/ directory
# - example_mimic.ipynb: Full MIMIC-IV extraction workflow
# - example_eicu.ipynb: eICU extraction workflow
```

## Architecture

### Core Components

**1. Configuration System (`src/open_icu/config/`)**
   - `source.py`: Defines data source configurations (SourceConfig, TableConfig, EventConfig)
   - `concept.py`: Medical concept definitions (currently a placeholder)
   - `utils.py`: YAML config loading utilities
   - Configuration files live in `configs/source/` (e.g., `mimic.yml`, `eicu.yml`)

**2. MEDS Processing (`src/open_icu/meds/`)**
   - `processor.py`: Core ETL logic - reads source data, performs joins, transforms to MEDS format
   - `processor_rs.py`: Rust-based processor (untracked file, likely in development)
   - `schema.py`: PyArrow schema definitions for MEDS data validation
   - `project.py`: MEDSProject class for managing output directory structure

### Configuration-Driven Architecture

The system uses a declarative YAML configuration approach:

1. **Source Configurations** (`configs/source/`): Define how to map raw ICU data tables to MEDS events
   - Specify tables, fields, data types, and joins
   - Define events to extract (e.g., medications, chart events, ICU stays)
   - Map source columns to MEDS schema fields (subject_id, time, code, numeric_value, text_value)
   - Support for calculated datetime fields and field constants

2. **Processing Flow**:
   - Load source config from YAML
   - Read source data tables using Dask (for scalability)
   - Apply field type conversions and datetime calculations
   - Perform table joins (e.g., joining item IDs with labels)
   - Extract events per configuration
   - Concatenate multi-field codes (e.g., "label//unit")
   - Write to Parquet files in MEDS format
   - Generate metadata (codes.parquet, dataset.json)

3. **MEDS Schema**:
   - Standard fields: subject_id, time, code, numeric_value, text_value
   - Extension fields: hadm_id, stay_id (hospital admission ID, ICU stay ID)
   - All data validated against PyArrow schema

### Key Design Patterns

- **Pydantic Models**: All configs use Pydantic for validation with computed fields
- **Dask DataFrames**: Used for parallel, out-of-core processing of large datasets
- **PyArrow Backend**: String data uses PyArrow for memory efficiency
- **Partition-based Processing**: map_partitions used for custom transformations
- **MEDS Compliance**: Outputs conform to MEDS v0.4.0+ standard

## Data Flow Example

For MIMIC-IV medications table:
1. Read `icu/inputevents.csv` (starttime, endtime, amount, rate, itemid)
2. Join with `icu/d_items.csv` (itemid → label)
3. Extract two events:
   - "dosage" event: endtime + label//amountuom → numeric_value=amount
   - "rate" event: starttime + label//rateuom → numeric_value=rate
4. Write to `output/data/mimic_medications_{event}_{partition}.parquet`
5. Accumulate unique codes to `output/metadata/codes.parquet`

## Important Conventions

### Code Style
- Line length: 120 characters (Ruff configured)
- Python version: 3.13+ (specified in .python-version)
- Use Ruff for formatting and linting (follows Black-compatible style)
- Type hints required: mypy strict mode (`disallow_untyped_defs = true`)
- String storage: Use PyArrow backend for pandas/dask string columns

### Testing
- Test discovery: `test_*.py` or `*_test.py` pattern
- Coverage required on src/ directory
- Deprecation warnings are filtered in pytest config
- Import mode: importlib (not prepend/append)

### Commit Convention

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aidh-ms/OpenICU](https://github.com/aidh-ms/OpenICU) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
