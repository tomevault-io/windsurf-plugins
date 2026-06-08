---
trigger: always_on
description: - **Unrelated changes:** Do not modify files unrelated to the current task without asking first.
---

# Agent Instructions

## General

- **Unrelated changes:** Do not modify files unrelated to the current task without asking first.
- **Destructive actions:** Always ask for approval before performing destructive or hard-to-reverse actions (e.g. `git push --force`, `git reset --hard`, deleting branches/files, dropping tables).

## Project Overview

This is a Home Assistant custom integration for NeoPool/VistaPool pool controllers connected via Modbus TCP. It lives under `custom_components/neopool/` and follows the standard HA integration pattern.

## Development Commands

```bash
# Install dev dependencies
pip install -r requirements-dev.txt

# Run all tests
pytest

# Run a single test file
pytest tests/test_sensor.py

# Run tests with coverage
pytest --cov=custom_components/vistapool --cov-report=term-missing tests/

# Type checking (must be 0 errors)
basedpyright

# Linting
ruff check

# Formatting check
ruff format --check

# Auto-fix formatting
ruff format
```

## Architecture

### Data Flow

```
Config Flow → ConfigEntry → NeoPoolCoordinator → NeoPoolModbusClient
                                    ↓
                         Platform entities subscribe
                         (sensor, switch, number, select, button, light, binary_sensor)
```

- **`modbus.py`** (`NeoPoolModbusClient`): Low-level Modbus TCP communication via `pymodbus`. Reads/writes registers, decodes raw register values into structured dicts.
- **`coordinator.py`** (`NeoPoolCoordinator`): `DataUpdateCoordinator` subclass. Polls `NeoPoolModbusClient` on `scan_interval`, distributes data to all platform entities. Also handles winter mode (suspends polling) and follow-up refresh after writes.
- **`const.py`**: Central definition file (~1200 lines). All entity definitions (keys, register addresses, device classes, units, options) live here as data structures. Adding a new entity usually means only editing `const.py`.
- **`entity.py`**: Base `NeoPoolEntity` — shared `unique_id`, `device_info`, `available` logic.
- **Platform files** (`sensor.py`, `switch.py`, etc.): Thin wrappers that read from `coordinator.data` using keys defined in `const.py`.

### Key Patterns

- Entity definitions in `const.py` are data-driven; platform files iterate over them to create entities. New entities rarely require changes outside `const.py`.
- `coordinator.data` is a flat `dict[str, Any]` keyed by the entity keys defined in `const.py`.
- Capability detection (hydrolysis, pH, Redox, chlorine, etc.) sets `CAPABILITY_KEYS` in coordinator data; entities check these to decide whether to register/show.
- `modbus_compat.py` abstracts pymodbus API differences between versions.
- `migration.py` handles config entry version upgrades (imported and re-exported from `__init__.py` for HA to discover).

## Branch Naming

Follow [Conventional Branch](https://conventional-branch.github.io/) format: `<type>/<description>`

- Lowercase alphanumerics and hyphens only (dots allowed in release versions)
- No consecutive, leading, or trailing hyphens or dots
- Include ticket/issue number when applicable

| prefix     | when to use                                 |
| ---------- | ------------------------------------------- |
| `feature/` | new feature (alias: `feat/`)                |
| `bugfix/`  | bug fix (alias: `fix/`)                     |
| `hotfix/`  | urgent fix                                  |
| `release/` | release preparation (e.g. `release/v1.2.0`) |
| `chore/`   | non-code tasks (deps, docs, config)         |

Examples: `feat/add-login-page`, `fix/header-bug`, `feature/issue-123-new-login`

## Git Commits

### Approval

- **Never commit automatically.** Always wait for my explicit approval before running `git commit`.
- **Tests:** If the project has tests, run them before proposing a commit. Verify that all tests pass and that code coverage has not decreased.

### Commit Message Format

Always use the format: `<type>(<scope>): <gitmoji> <description>`

**Rules:**

- `scope` is optional but use it when the change is clearly scoped to a module
  (e.g. `sensor`, `binary_sensor`, `button`, `light`, `number`, `select`, `switch`, `modbus`, `config`, `coordinator`, `entity`, `diagnostics`, `helpers`)
- `description`: lowercase, imperative mood ("add", not "added"), no period at end

**Pick the type and gitmoji that best reflect the nature of the change:**

| type       | gitmoji | when to use                                        |
| ---------- | ------- | -------------------------------------------------- |
| `feat`     | ✨      | new user-facing feature                            |
| `feat!`    | 💥      | breaking change                                    |
| `fix`      | 🐛      | bug fix                                            |
| `fix`      | 🩹      | minor / non-critical fix (style, typo, off-by-one) |
| `fix`      | 🚑️      | critical hotfix                                    |
| `fix`      | 🔒️      | security / privacy fix                             |
| `docs`     | 📝      | add or update documentation or comments            |
| `style`    | 🎨      | code structure / formatting (no logic change)      |
| `style`    | 💄      | UI or style files                                  |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [svasek/homeassistant-neopool-modbus](https://github.com/svasek/homeassistant-neopool-modbus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-08 -->
