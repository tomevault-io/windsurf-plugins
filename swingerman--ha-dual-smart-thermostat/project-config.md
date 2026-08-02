---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Home Assistant Dual Smart Thermostat - An enhanced thermostat component supporting multiple HVAC modes (heating, cooling, heat pump, fan, humidity control), advanced features (floor temperature control, window/door sensors, presets), and sophisticated control logic.

**Target**: Home Assistant 2025.1.0+
**Language**: Python 3.13

## Essential Commands

### Development with Docker (Recommended)

**IMPORTANT: For Claude Code development, always use Docker scripts for testing and linting to ensure consistent environment and avoid local Python dependency issues.**

The project provides convenient Docker scripts in the `scripts/` folder:

```bash
# Testing - Use docker-test for all test runs
./scripts/docker-test                              # Run all tests
./scripts/docker-test tests/test_heater_mode.py    # Run specific test file
./scripts/docker-test -k "heater"                  # Run tests matching pattern
./scripts/docker-test --cov                        # Run with coverage report
./scripts/docker-test --log-cli-level=DEBUG        # Run with debug logging

# Linting - Use docker-lint for all code quality checks (REQUIRED before commit)
./scripts/docker-lint                              # Check all linting (isort, black, flake8, codespell, ruff)
./scripts/docker-lint --fix                        # Auto-fix linting issues

# Interactive Shell - For debugging and exploration
./scripts/docker-shell                             # Open bash shell in container
./scripts/docker-shell python                      # Open Python REPL in container
```

**Why use Docker scripts:**
- Guaranteed consistent Python 3.13 + HA 2025.1.0+ environment
- No local dependency conflicts or version mismatches
- Same environment as CI/CD pipeline
- Automatic image building if needed
- Live source code mounting (changes reflected immediately)

### Local Development (Alternative)

If you prefer local development without Docker:

```bash
# Install dependencies
pip install -r requirements-dev.txt
pre-commit install

# Testing (local alternative)
pytest                                    # Run all tests
pytest tests/test_heater_mode.py          # Run specific test file
pytest --log-cli-level=DEBUG              # Run with debug logging

# Linting (local alternative - ALL must pass before commit)
isort . --check-only --diff               # Import sorting
black --check .                           # Code formatting
flake8 .                                  # Style/linting
codespell                                 # Spell checking
ruff check .                              # Additional linting

# Auto-fix linting issues (local)
isort .
black .
ruff check . --fix
```

### Advanced Docker Usage

```bash
# Build with specific Home Assistant version
HA_VERSION=2025.2.0 docker-compose build dev
HA_VERSION=latest docker-compose build dev

# Run custom commands in container
docker-compose run --rm dev <command>
```

### Code Quality Requirements

**ALL code MUST pass linting checks before commit:**
- `isort` - Import sorting
- `black` - Code formatting (88 character line length)
- `flake8` - Style/linting
- `codespell` - Spell checking
- `ruff` - Additional linting

**Run `./scripts/docker-lint` before committing. GitHub workflows will reject failing commits.**

## Architecture Overview

### Modular Design Pattern

The codebase uses a **separation of concerns** architecture with distinct layers:

1. **Device Layer** (`hvac_device/`) - Hardware abstraction for different HVAC equipment types
2. **Manager Layer** (`managers/`) - Shared business logic (features, state, environment)
3. **Controller Layer** (`hvac_controller/`) - Orchestration between devices and managers
4. **Climate Entity** (`climate.py`) - Home Assistant integration interface

### Core Components

#### Device Types (`hvac_device/`)
Abstraction layer for different HVAC equipment:
- `heater_device.py` - Basic heating
- `cooler_device.py` - Air conditioning
- `heat_pump_device.py` - Combined heating/cooling (single switch)
- `heater_cooler_device.py` - Dual heating/cooling (separate switches)
- `heater_aux_heater_device.py` - Two-stage heating
- `fan_device.py` - Fan-only operation
- `dryer_device.py` - Humidity control
- `hvac_device_factory.py` - **Factory pattern** creates appropriate device based on configuration

#### Managers (`managers/`)
Shared logic components handling specific responsibilities:
- `state_manager.py` - Persistence and state restoration
- `environment_manager.py` - Environmental condition tracking (temperature, humidity, sensors)
- `feature_manager.py` - Feature enablement and configuration
- `opening_manager.py` - Window/door sensor handling
- `preset_manager.py` - Preset mode management
- `hvac_power_manager.py` - Power cycling and keep-alive logic

#### Controllers (`hvac_controller/`)
Orchestration of control logic:
- `generic_controller.py` - Base controller with common logic
- `heater_controller.py` - Heating-specific control
- `cooler_controller.py` - Cooling-specific control
- `hvac_controller.py` - Top-level coordinator

#### HVAC Action Reasons (`hvac_action_reason/`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swingerman/ha-dual-smart-thermostat](https://github.com/swingerman/ha-dual-smart-thermostat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
