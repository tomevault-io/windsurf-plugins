---
trigger: always_on
description: This document provides guidance for AI coding agents working on this Home Assistant custom integration project.
---

# AI Agent Instructions - IEC Custom Component

This document provides guidance for AI coding agents working on this Home Assistant custom integration project.

## Project Overview

This is a Home Assistant custom component for the Israel Electric Corporation (IEC) API. It provides sensors and binary sensors for monitoring electricity consumption, invoices, and meter readings.

**Integration details:**
- **Domain:** `iec`
- **Title:** IEC (Israel Electric Corporation)
- **Repository:** GuyKh/iec-custom-component

**Key directories:**
- `custom_components/iec/` - Main integration code
- `config/` - Home Assistant configuration for local testing
- `.github/workflows/` - CI/CD workflows

## Tech Stack

- **Python**: 3.13+
- **Home Assistant**: 2025.10.2+
- **iec-api**: 0.5.15 (Python client for IEC API)
- **Linting**: ruff (with Home Assistant rules)
- **Type Checking**: mypy

## Code Structure

```
custom_components/iec/
├── __init__.py          # Integration entry point (async_setup_entry, async_unload_entry)
├── const.py             # Constants and DOMAIN definitions
├── config_flow.py       # UI configuration flow
├── coordinator.py       # DataUpdateCoordinator - main API logic
├── commons.py           # Shared utilities (timezone, helpers)
├── sensor.py            # Sensor platform
├── binary_sensor.py     # Binary sensor platform
├── iec_entity.py        # Base entity class
└── services.yaml        # Service definitions
```

## Local Development

**Always use the project's scripts** — do NOT craft your own `hass`, `pip`, or similar commands. The scripts handle environment setup correctly.

**Setup:**
```bash
./scripts/setup  # Install dependencies
```

**Start Home Assistant:**
```bash
./scripts/develop  # Start HA development environment
```

**Debugging:**
Enable debug logging in `config/configuration.yaml`:
```yaml
logger:
  default: info
  logs:
    custom_components.iec: debug
    iec_api: debug
```

**Reading logs:**
- Terminal where `./script/develop` runs
- `config/home-assistant.log`

## Workflow

### Starting New Work

**When starting a new task, always ask the user first:**
- Should I switch to `main` branch and rebase?
- Or should I work from the current branch?

Then checkout a new feature branch before beginning work. Never work directly on `main` or stale branches.

### Branch Naming Convention
- Features: `feature/description`
- Bug fixes: `fix/description`
- Documentation: `docs/description`

## Code Style

**Python:**
- 4 spaces indentation
- 120 character lines
- Double quotes for strings
- Full type hints (mypy strict)
- Async for all I/O operations
- Follow ruff rules from `.ruff.toml`

**Validation commands:**
```bash
./scripts/lint        # Run ruff linter (auto-fixes where possible)
```

## Key Patterns

### Integration Setup
- Uses `ConfigEntry` for UI-based configuration
- Supports multiple accounts (one entry per account)
- Registers `PLATFORMS`: `sensor` and `binary_sensor`

### Coordinator Pattern
- `IecApiCoordinator` extends `DataUpdateCoordinator`
- Handles all API calls to IEC
- Manages JWT tokens and authentication
- Stores data in `hass.data[DOMAIN][entry.entry_id]`
- Entities → Coordinator → API Client (never skip layers)
- Raise `ConfigEntryAuthFailed` (triggers reauth) or `UpdateFailed` (retry)

### Entity Pattern
- Base `IecEntity` class in `iec_entity.py`
- Entities inherit from `CoordinatorEntity[IecApiCoordinator]`
- Read from `coordinator.data`, never call API directly
- Use `EntityDescription` dataclasses for static entity metadata

### Config Flow
- Implement in `config_flow.py`
- Support user setup, reauth
- Always set `unique_id` for entries

## Project-Specific Rules

### IEC-Specific Identifiers
- **Domain:** `iec`
- **Class prefix:** `Iec`

### API Concepts
- **Contracts**: Electrical contracts linked to user account
- **Meters**: Physical meters associated with contracts
- **Invoices**: Billing information (consumption, amount, payment status)
- **Meter Readings**: Daily/hourly consumption data from smart meters
- **Future Consumption**: Predicted consumption based on historical data

### Constants
- All constants defined in `const.py`
- Use `DOMAIN = "iec"` for all domain references
- Prefix IEC-specific classes with `Iec`

### Data Storage Keys (from `const.py`)
- `JWT_DICT_NAME` = "jwt"
- `STATIC_DICT_NAME` = "statics"
- `ATTRIBUTES_DICT_NAME` = "entity_attributes"
- `ESTIMATED_BILL_DICT_NAME` = "estimated_bill"
- `INVOICE_DICT_NAME` = "invoice"
- `CONTRACT_DICT_NAME` = "contract"
- `DAILY_READINGS_DICT_NAME` = "daily_readings"
- `FUTURE_CONSUMPTIONS_DICT_NAME` = "future_consumption"

## Common Tasks

### Adding a New Sensor
1. Add constants to `const.py` if needed
2. Add sensor logic in `sensor.py`
3. Follow existing sensor patterns (base entity → specific sensor)
4. Add full type annotations (mypy strict)
5. Use `EntityDescription` dataclass for static metadata

### Adding a New Binary Sensor
1. Add constants to `const.py` if needed
2. Add binary sensor logic in `binary_sensor.py`
3. Follow existing patterns
4. Add full type annotations

### API Changes
When iec-api updates:
1. Update version in `requirements.txt`
2. Run type check to find breaking changes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GuyKh/iec-custom-component](https://github.com/GuyKh/iec-custom-component) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
