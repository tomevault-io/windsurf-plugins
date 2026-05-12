---
trigger: always_on
description: This document provides essential guidelines for AI agents working on this codebase.
---

# CLAUDE.md - AI Agent Guidelines

This document provides essential guidelines for AI agents working on this codebase.

## Critical Context

**This code controls heating for real homes.** Mistakes can result in:
- Pipes freezing and bursting (costly property damage)
- Excessive energy bills
- Uncomfortable living conditions
- Hardware damage to valves and boilers

**Quality and correctness are paramount.** When in doubt, ask questions rather than make assumptions.

## Environment Setup

**ALWAYS run before any code quality checks:**

```bash
uv sync --dev
```

This installs all dependencies including dev extras (pytest, ruff, ty).

## Pre-Commit Checklist

**BEFORE committing any changes, run ALL of these checks in order:**

```bash
# 1. Run tests first - ensures code works correctly
uv run pytest

# 2. Format code with ruff (auto-fixes formatting issues)
uv run ruff format .

# 3. Lint with ruff (auto-fixes what it can)
uv run ruff check . --fix

# 4. Type check with ty
uv run ty check
```

**All checks must pass before committing.** CI will reject PRs that fail any of these.

## Pre-PR Checklist

**BEFORE creating a pull request, validate that your changes have 100% test coverage:**

```bash
# 1. Generate coverage XML report
uv run pytest --cov=custom_components/ufh_controller --cov-branch --cov-report=xml

# 2. Check diff coverage against main branch (100% required)
uv run diff-cover coverage.xml --compare-branch=main --fail-under=100
```

**If diff coverage fails:**
1. Review the output to see which specific lines lack coverage
2. Add tests for the uncovered lines
3. Re-run the diff coverage check until it passes

**Why this matters:** Codecov enforces diff coverage on PRs. Validating locally prevents wasted reviewer time on PRs that will fail CI.

## Project Structure

```
custom_components/ufh_controller/
├── __init__.py          # Entry point, async_setup_entry
├── coordinator.py       # DataUpdateCoordinator (main control loop)
├── config_flow.py       # UI configuration flows
├── const.py             # Constants, defaults, enums
├── data.py              # Custom types (UFHControllerConfigEntry, UFHControllerData)
├── device.py            # Device info helpers
├── entity.py            # Base entity classes (UFHControllerEntity, UFHControllerZoneEntity)
├── recorder.py          # Recorder query helpers (state averages, window detection)
├── core/
│   ├── controller.py    # Control logic orchestration
│   ├── zone.py          # Zone state and decision logic
│   ├── pid.py           # PID controller implementation
│   ├── history.py       # Observation period datetime helpers
│   ├── ema.py           # EMA filter for temperature smoothing
│   ├── heating_curve.py # Heating curve for outdoor temp compensation
│   └── hysteresis.py    # Hysteresis rounding for display flicker prevention
├── climate.py           # Climate entity platform
├── sensor.py            # Sensor entities (duty cycle, PID values)
├── binary_sensor.py     # Binary sensors (blocked, heat request)
├── select.py            # Mode selector entity
└── switch.py            # Switch entities (flush enabled)

tests/                   # Test suite (see Test Organization below)
│   ├── unit/            # Pure logic tests, no HA dependencies
│   ├── integration/     # Entity platform tests with mocked HA
│   ├── scenarios/       # End-to-end workflows and resilience
│   └── config/          # Config flows and setup lifecycle
docs/                    # Documentation (see docs/index.md for full list)
```

## Documentation Synchronization

**The `docs/` directory is the source of truth for design decisions.** See `docs/index.md` for the full documentation structure.

When making changes:
1. Check if your changes align with the documentation
2. If changes conflict with the docs, update BOTH the code AND relevant doc files
3. Never leave the documentation out of sync with the implementation
4. Document any new features, entities, or configuration options in the appropriate doc file

## Testing Requirements

### Test Coverage
- **Overall**: 90% minimum (enforced in pyproject.toml and CI)
- **Core Modules**: 100% target, 98% minimum acceptable (critical control logic)

### Bug Fixes: Reproduce First
When fixing bugs:
1. **Write a failing test case first** that reproduces the bug
2. Verify the test fails as expected
3. Implement the fix
4. Verify the test now passes
5. Add any additional edge case tests

This ensures bugs don't regress and documents the expected behavior.

### New Features: Test Thoroughly
- Write tests for all new functionality
- Cover edge cases (null values, boundary conditions, error states)
- Test integration with Home Assistant entities where applicable

### Test Fixtures
Common fixtures are in `tests/conftest.py`:
- `mock_config_entry` - Config entry with one zone
- `mock_config_entry_no_zones` - Config entry without zones
- `mock_config_entry_multiple_zones` - Config entry with two zones
- `mock_config_entry_with_heat_request` - Config entry with heat request entity
- `mock_config_entry_all_entities` - Config entry with all controller-level entities
- `mock_config_entry_with_supply_temp` - Config entry with supply temp entity

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lnagel/hass-ufh-controller](https://github.com/lnagel/hass-ufh-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
