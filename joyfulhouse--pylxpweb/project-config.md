---
trigger: always_on
description: **Last Updated**: 2026-02-05
---

# CLAUDE.md - Development Guidelines for pylxpweb

**Last Updated**: 2026-02-05
**Version**: 0.6.5
**Purpose**: Guide Claude Code development in this repository

## Project Overview

**pylxpweb** is a Python client library for Luxpower/EG4 solar inverters and energy storage systems. It provides programmatic access to the Luxpower/EG4 web monitoring API with a focus on type safety, async operations, and Home Assistant integration readiness.

**Key Features**:
- Complete API coverage (authentication, discovery, runtime data, energy stats, control operations)
- Object-oriented device hierarchy (Station → ParallelGroup → Inverter → BatteryBank → Battery)
- 100% feature parity with EG4 Web Monitor HA integration
- Async/await throughout with concurrent operations
- Type-safe with mypy strict mode (Pydantic v2 models)
- Production-ready: 288+ tests, >81% coverage, zero linting errors

**Repository**: `joyfulhouse/pylxpweb`

## Quick Reference

### Device Hierarchy
```
Station (Plant)
├── Parallel Group (0-N) - Multi-inverter configurations
│   ├── MID Device (GridBOSS) - Optional, 0-1 per group
│   └── Inverters (1-N)
│       └── Battery Bank - Aggregate battery data
│           └── Batteries (0-N) - Individual battery modules
└── Standalone Inverters (0-N) - Single inverter setups
```

### Regional API Endpoints
| Region | Base URL | Use For |
|--------|----------|---------|
| US (EG4) | `https://monitor.eg4electronics.com` | EG4-branded devices (default) |
| US (Luxpower) | `https://us.luxpowertek.com` | Luxpower-branded devices (US) |
| Americas (Luxpower) | `https://na.luxpowertek.com` | Brazil, Latin America |
| EU (Luxpower) | `https://eu.luxpowertek.com` | Luxpower-branded devices (EU) |
| Asia Pacific (Luxpower) | `https://sea.luxpowertek.com` | Southeast Asia, Australia |
| Middle East & Africa (Luxpower) | `https://af.luxpowertek.com` | Middle East, Africa |
| China (Luxpower) | `https://server.luxpowertek.com` | China mainland |

### Data Scaling (CRITICAL)
| Type | Scaling | Example |
|------|---------|---------|
| Voltage (Inverter) | ÷100 | 5100 → 51.00V |
| Voltage (Battery Bank) | ÷10 | 539 → 53.9V |
| Voltage (Individual Battery) | ÷100 | 5394 → 53.94V |
| Voltage (Cell) | ÷1000 | 3364 → 3.364V |
| Current | ÷100 | 1500 → 15.00A |
| Frequency | ÷100 | 5998 → 59.98Hz |
| Temperature | Direct | 39 → 39°C |
| Power | Direct | 1030 → 1030W |
| Energy (API) | ÷10 | 184 → 18.4 kWh |

**WARNING**: Note different voltage scaling for battery bank vs individual batteries!

### Device Type Identification

There are two different "device type" concepts:

| Concept | Description | Values |
|---------|-------------|--------|
| **API `deviceType`** | Web API category for routing | 6=Inverter, 9=GridBOSS |
| **`HOLD_DEVICE_TYPE_CODE`** | Firmware model identifier (register 19) | Varies per model |

**Device Type Code Mapping:**
| Code | Family | Example Models |
|------|--------|----------------|
| 54 | SNA Series | SNA12K-US (split-phase, US) |
| 2092 | PV Series | 18KPV (high-voltage DC, US) |
| 12 | LXP-EU Series | LXP-EU 12K (European) |

**Feature Detection:**
```python
# Detect features based on inverter model
await inverter.detect_features()

# Check capabilities
inverter.supports_split_phase      # SNA series only
inverter.supports_volt_watt_curve  # PV/EU series
inverter.supports_parallel         # PV/EU series
inverter.model_family             # InverterFamily enum
inverter.device_type_code         # Raw code (54, 2092, 12)
```

See [docs/DEVICE_TYPES.md](docs/DEVICE_TYPES.md) for comprehensive documentation.

## Development Standards

### Code Quality Requirements
1. **Type Hints**: All functions must have complete type annotations
   - Use `from __future__ import annotations` for forward references
   - mypy strict mode enforced (`mypy src/pylxpweb/ --strict`)

2. **Async/Await**: All I/O operations must be async
   - Use `aiohttp` for HTTP requests
   - Use `asyncio.gather()` for concurrent operations
   - No blocking operations in async code

3. **Error Handling**:
   - Custom exceptions in `src/pylxpweb/exceptions.py`
   - Specific types: `AuthenticationError`, `ConnectionError`, `LuxpowerAPIError`
   - Proper exception hierarchy with contextual error messages

4. **Testing**:
   - Target: >90% code coverage (currently >81%)
   - Use `pytest` with `pytest-asyncio`
   - Mock external API calls in unit tests (`tests/unit/`)
   - Real API tests in `tests/integration/` (requires credentials)

5. **Code Style**:
   - Format: `ruff check --fix && ruff format`
   - Lint: `ruff check src/ tests/`
   - Google-style docstrings for all public APIs

### Pre-Commit Workflow (REQUIRED)
Before any commit, automatically run:
```bash
# 1. Format and lint
ruff check --fix && ruff format

# 2. Type checking
mypy src/pylxpweb/ --strict

# 3. Tests with coverage
pytest tests/unit/ --cov=pylxpweb --cov-report=term-missing

# All checks must pass before committing
```

## API Architecture

### Key Endpoints (Complete List)
**Authentication**:
- `POST /WManage/api/login` - Session duration: ~2 hours

**Discovery**:
- `POST /WManage/web/config/plant/list/viewer` - List stations
- `POST /WManage/api/inverterOverview/getParallelGroupDetails` - Device hierarchy (requires GridBOSS serial)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joyfulhouse/pylxpweb](https://github.com/joyfulhouse/pylxpweb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-20 -->
