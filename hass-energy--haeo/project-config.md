---
trigger: always_on
description: This repository contains **HAEO** (Home Assistant Energy Optimizer) - a Python 3.13+ Home Assistant custom component for energy network optimization using linear programming.
---

# HAEO agent instructions

This repository contains **HAEO** (Home Assistant Energy Optimizer) - a Python 3.13+ Home Assistant custom component for energy network optimization using linear programming.

## Project overview

HAEO optimizes energy usage across battery storage, grid import/export, loads, and generators using linear programming.
The integration provides real-time optimization based on energy prices, forecasts, and system constraints.

### Core components

The integration follows a layered architecture:

- **Model layer** (`core/model/`): LP formulation with elements, constraints, and cost functions
- **Elements layer** (`elements/`, `core/schema/`, `core/adapters/`): Element registry, schemas, and adapters
- **Coordinator** (`coordinator/`): Orchestrates data loading, optimization, and result extraction
- **Entities** (`sensor.py`, `number.py`, `switch.py`, `entities/`): Expose inputs and optimization results to Home Assistant
- **Config flows** (`flows/`): Subentry-based configuration for hub and elements

See [architecture guide](docs/developer-guide/architecture.md) for detailed component interactions.

### Project structure

```
custom_components/haeo/     # Home Assistant integration
├── core/                   # Core infrastructure (no HA dependencies)
│   ├── model/              # LP model (constraints, variables, optimization)
│   ├── data/               # Data loading utilities and forecast extractors
│   ├── schema/             # Element schemas, field types, sections, migrations
│   └── adapters/           # Element adapters and policy compilation
├── elements/               # Element registry, availability, input field derivation
├── flows/                  # Hub, options, and element config flows
│   └── elements/           # Per-element flow implementations
├── coordinator/            # Optimization cycle and network assembly
├── entities/               # Entity classes behind the sensor, number, and switch platforms
├── horizon.py              # Forecast time windows
├── input_stores.py         # Input store layer backing the input entities
└── translations/           # i18n strings (en.json)
tools/                      # Developer CLIs (check, diag, sim, snapshot)
tests/scenarios/            # End-to-end scenario tests
docs/                       # Documentation
```

Tests are colocated with source code in `tests/` subdirectories within each package.
The top-level `tests/` directory holds only scenario tests, guide tests, and CLI tool tests.
`custom_components/haeo/sensors/` contains tests, not an implementation.

The `core/` package must never import Home Assistant; import-linter enforces this.

## Development tools

- **Package manager**: uv (use `uv sync` for dependencies, `uv run` to execute tools)
- **Testing**: pytest (scenarios require `-m scenario` marker and are skipped in CI)
- **Linting/Formatting**: Ruff (Python), Prettier (JSON), mdformat (Markdown)
- **Type checking**: Pyright

## Agent behavioral rules

These rules apply to all AI agent interactions with this codebase:

### Design principles

**Convention over configuration**: Prefer uniform patterns that work the same everywhere over configurable options that require case-by-case logic.
When code paths diverge based on metadata flags or configuration, ask whether the divergence is necessary.
Often, a single convention that handles all cases uniformly is simpler and more maintainable.

- Derive behavior from existing structure rather than adding metadata flags
- Make all instances of a pattern work the same way - no special cases
- Let upstream validation (e.g., config flows) enforce constraints so downstream code can assume valid data
- Config flows use `vol.Required()` and `vol.Optional()` to enforce required fields at entry time
- Downstream code (coordinator, adapters) can assume required fields are present because config flow guarantees it
- For optional values, if they are missing or None, skip them uniformly throughout processing

**Composition over complexity**: Build features by composing simple, focused components rather than adding conditional logic to existing code.
Each component should do one thing well without needing to know about the internals of other components.

- Separate concerns: validation happens at config flow boundaries, processing assumes valid input
- Avoid "check if X then do Y else do Z" patterns - instead, make X and Y go through the same code path
- When adding a feature, prefer creating new simple components over adding branches to existing ones
- Runtime code uses the result of schema validation, not the schema itself - the schema's job is done at configuration time

### Clean changes

When making changes, don't leave behind comments describing what was once there.
Comments should always describe code as it exists without reference to former code.

### Commit messages

Use plain English commit messages without conventional commit prefixes (`feat:`, `fix:`, `refactor:`, etc.).
Write a short summary line in imperative mood, followed by a blank line and bullet points if needed.

### API evolution


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hass-energy/haeo](https://github.com/hass-energy/haeo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
