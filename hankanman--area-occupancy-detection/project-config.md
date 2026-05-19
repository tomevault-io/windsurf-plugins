---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Area Occupancy Detection is a Home Assistant custom integration that uses Bayesian probability to intelligently detect room occupancy. It combines multiple sensor inputs (motion, media devices, appliances, environmental sensors) with learned historical patterns to provide probabilistic occupancy detection.

The integration learns from your patterns over time, uses decay functions to handle stationary occupancy, and provides both binary occupancy status and probability sensors that automations can use.

## Development Commands

### Environment Setup

```bash
# Bootstrap environment (installs uv, creates venv, installs dependencies, sets up pre-commit)
scripts/bootstrap

# Manual setup if not using devcontainer
# 1. Install uv: curl -LsSf https://astral.sh/uv/install.sh | sh
# 2. Run: scripts/bootstrap
# 3. Activate: source .venv/bin/activate
```

### Code Quality

```bash
# Lint and format code (runs ruff format and ruff check --fix)
scripts/lint

# Manual linting
uv run ruff format .
uv run ruff check . --fix
```

### Testing

```bash
# Run all tests with coverage report
scripts/test

# Run specific test file
uv run pytest tests/test_area_area.py

# Run with verbose output
uv run pytest -v

# Run specific test within a file
uv run pytest tests/test_area_area.py::test_area_initialization -v
```

### Development Environment

This project uses a **devcontainer** that provides a standalone Home Assistant instance. When opening in VS Code, accept the devcontainer prompt to get:
- Pre-configured development environment
- Home Assistant running with `config/configuration.yaml`
- All dependencies installed
- Pre-commit hooks configured

## Architecture

### High-Level Structure

The integration uses a **single-instance coordinator architecture** that manages multiple areas:

```
AreaOccupancyCoordinator (global singleton)
├── AreaOccupancyDB (SQLite database with SQLAlchemy)
├── IntegrationConfig (global settings)
└── areas: dict[str, Area]
    └── Area (per-room instance)
        ├── AreaConfig (sensors, weights, thresholds)
        ├── EntityManager (tracks sensor states and evidence)
        ├── Prior (learned probabilities)
        └── Purpose (room type and decay settings)
```

### Key Concepts

**Multi-Area Management**: A single coordinator manages all configured areas. Each area has its own device in Home Assistant with associated entities (sensors, binary sensors, numbers).

**Entity Types**: Sensors are classified by `InputType` (MOTION, MEDIA, APPLIANCE, DOOR, WINDOW, ENVIRONMENTAL, POWER, WASP). Each type has different probability contributions and weights.

**Bayesian Calculation**: The core algorithm combines:
- **Prior probabilities**: Learned from historical patterns (time-of-day, day-of-week)
- **Sensor evidence**: Current state of all sensors with type-specific weights
- **Decay**: Gradual probability reduction when no new evidence arrives
- Result: A probability (1-99%) that updates continuously

**Database Architecture**: SQLite with SQLAlchemy ORM, organized in modules:
- `db/core.py`: Database initialization and session management
- `db/schema.py`: SQLAlchemy table definitions (Areas, Entities, Intervals, Aggregates, etc.)
- `db/operations.py`: CRUD operations for entities and intervals
- `db/aggregation.py`: Time-series aggregation (hourly, daily, weekly, monthly)
- `db/correlation.py`: Statistical correlation analysis between sensors and occupancy
- `db/queries.py`: Complex queries for occupied intervals and cache management
- `db/sync.py`: Import entity states from Home Assistant recorder
- `db/maintenance.py`: Health checks, pruning, backups

**Analysis Pipeline**: Every hour (configurable), the coordinator runs:
1. Sync states from recorder → import recent entity state changes
2. Health check and pruning → validate database integrity, remove old data
3. Populate occupied intervals cache → identify when areas were occupied
4. Run aggregations → hourly/daily/weekly/monthly summaries
5. Recalculate priors → update learned probabilities from historical data
6. Correlation analysis → identify sensor relationships with occupancy
7. Save and refresh → persist changes, update entities

### Critical Files

- `coordinator.py`: Main coordinator managing lifecycle, timers, and multi-area orchestration
- `area/area.py`: Per-area logic, encapsulates configuration, entities, priors, and calculations
- `data/entity.py`: Entity tracking, state management, evidence detection (380+ lines)
- `data/analysis.py`: Orchestrates the full analysis pipeline
- `data/prior.py`: Prior probability calculations from historical patterns
- `data/config.py`: Configuration management for both integration-level and area-level settings
- `data/decay.py`: Time-based probability decay implementation
- `data/purpose.py`: Room purpose definitions (social, work, sleep, etc.) with default decay settings
- `db/core.py`: Database initialization, connection management
- `db/correlation.py`: Statistical analysis of sensor-occupancy relationships (660+ lines)
- `utils.py`: Bayesian probability calculations, state mapping utilities


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hankanman/Area-Occupancy-Detection](https://github.com/Hankanman/Area-Occupancy-Detection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
