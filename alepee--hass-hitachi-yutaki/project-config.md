---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Home Assistant custom integration for Hitachi air-to-water heat pumps (Yutaki and Yutampo models). Communicates via Modbus with ATW-MBS-02 and HC-A(16/64)MB gateways. Follows hexagonal architecture. Version is in `manifest.json`. See [`docs/`](docs/) for detailed documentation.

## Development Commands

All commands are available as `make` targets. Run `make help` to list them.

```bash
make setup          # Full project setup (deps + pre-commit hooks)
make install        # Install/reinstall all dependencies
make check          # Run all code quality checks (lint + format)
make lint           # Run ruff linter with auto-fix
make test           # Run all tests
make test-domain    # Run domain layer tests only (pure Python, no HA)
make test-coverage  # Run tests with coverage report
make ha-run         # Start a local HA dev instance with debug config
make bump           # Bump version (patch by default)
make bump PART=minor # Bump minor version (e.g., 2.0.2 → 2.1.0)
make bump PART=major # Bump major version (e.g., 2.1.0 → 3.0.0)
```

Pre-commit hooks are automatically installed by `make setup` and run ruff on every commit.

## Architecture

This integration follows **Hexagonal Architecture** (Ports and Adapters) with strict separation of concerns. See [docs/architecture.md](docs/architecture.md) for full architecture documentation.

### Critical Architecture Rules

**Domain Layer** (`domain/`):
- **NEVER** import `homeassistant.*`
- **NEVER** import from `adapters.*` or `entities.*`
- **NEVER** use external libraries (stdlib only)
- **ALWAYS** use Protocols for dependencies
- Pure business logic that can be tested without HA mocks

**Adapters Layer** (`adapters/`):
- Implements domain ports/protocols
- Bridges domain with Home Assistant infrastructure
- Delegates business logic to domain services
- Handles HA-specific concerns (state retrieval, entity data)

**Entity Layer** (`entities/`):
- Organized by **business domain** (not by entity type)
- Each domain has builder functions that return entity lists
- Uses base classes from `entities/base/` for common HA entity patterns
- Platform files (`sensor.py`, etc.) call builders and register entities

## Key Domain Concepts

See [docs/reference/domain-services.md](docs/reference/domain-services.md) for detailed service documentation.

### Heat Pump Profiles
- Auto-detected based on Modbus data; defines capabilities (DHW, pool, circuits, compressors)
- Base class: `HitachiHeatPumpProfile` with detection logic

### COP Calculation
- Coefficient of Performance monitoring using energy accumulation over time
- Quality levels: `no_data`, `insufficient_data`, `preliminary`, `optimal`

### Thermal Energy Tracking
- Separate tracking for heating and cooling (real-time power, daily energy, total energy)
- **Defrost filtering** and **post-cycle lock** prevent measurement noise

### Anonymous Telemetry
- Binary consent (Off / On) stored in `entry.options["telemetry_level"]`
- **Package** (`telemetry/`): models, collector (circular buffer), aggregator (daily stats from points), anonymizer (SHA-256 hash, 0.5°C rounding, geolocation rounding), HTTP client (gzip + retry), noop client
- **Coordinator wiring**: collector.collect() on each poll, 5-min flush timer, daily stats at day boundary, one-time installation info + register snapshot (fire-and-forget with `asyncio.Lock` + exponential backoff)
- **Backend**: Cloudflare Worker (ingestion/validation/rate-limit per payload type) → R2 (permanent JSON archive, partitioned Hive-style, all types). Installation payloads are also mirrored to Workers Analytics Engine (dataset `hitachi_installations`) for a Grafana fleet-inventory dashboard. Integration re-sends installation daily to keep WAE's 90-day window populated.
- **Consent flows**: options flow step (after sensors), repair flow for existing users (defaults to "on")
- **Diagnostic entity**: `sensor.telemetry_status` (ENUM: off/on) with attributes (points_buffered, last_send, send_failures)
- See [docs/reference/telemetry.md](docs/reference/telemetry.md)

### Devices Created
- **Gateway**, **Control Unit**, **Primary Compressor** (always present)
- **Secondary Compressor** (S80 only), **Circuit 1 & 2**, **DHW**, **Pool** (if configured)

## Important Development Notes

### When Adding New Entities

Follow the domain builder pattern. See [docs/development/adding-entities.md](docs/development/adding-entities.md) for the step-by-step guide. **Never** add business logic to entity classes.

### When Modifying Calculations

Domain logic goes in `domain/services/`, adapter logic in `adapters/calculators/`. See [docs/reference/domain-services.md](docs/reference/domain-services.md). Domain layer must remain HA-agnostic.

### Modbus Register Access

**Always read from STATUS registers** for sensor entities -- CONTROL registers only reflect what was commanded, not the actual running state. See [API Layer & Data Keys](docs/development/api-data-keys.md) for details.

### Circuit Climate Architecture

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alepee/hass-hitachi_yutaki](https://github.com/alepee/hass-hitachi_yutaki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
