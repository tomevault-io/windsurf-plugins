---
trigger: always_on
description: This is a **Home Assistant custom integration** for National Grid utility accounts. The repository is ~540KB with 79 files and ~1,400 lines of Python code in the integration. It's based on the `ludeeus/integration_blueprint` template.
---

# Copilot Instructions for ha_nationalgrid

## Project Overview

This is a **Home Assistant custom integration** for National Grid utility accounts. The repository is ~540KB with 79 files and ~1,400 lines of Python code in the integration. It's based on the `ludeeus/integration_blueprint` template.

**Key technologies:**
- **Language**: Python 3.13+ (required for Home Assistant 2025.2.4+)
- **Framework**: Home Assistant custom component
- **API Library**: `aionatgrid==0.4.0` (from https://github.com/ryanmorash/aionatgrid)
- **Distribution**: HACS (Home Assistant Community Store)
- **Linter/Formatter**: Ruff (version 0.14.14)
- **Testing**: pytest with pytest-homeassistant-custom-component

## Build & Validation Commands

### Essential Commands (Always Run These)

**IMPORTANT: Python 3.13+ is required.** If Python 3.12 or earlier is available, many commands will fail with dependency resolution errors because Home Assistant 2025.2.4+ requires Python >=3.13.0.

#### 1. Install Dependencies
```bash
python3 -m pip install -r requirements.txt
# OR use the helper script:
scripts/setup
```
**When to run**: First time setup, after any changes to requirements.txt, or after any dependency errors.

#### 2. Lint Code (REQUIRED before commit)
```bash
# Using script (recommended):
scripts/lint

# OR manually:
python3 -m ruff format .
python3 -m ruff check . --fix
```
**When to run**: Always run before committing code changes. The CI workflow will fail if linting fails.

**Expected output**: 
- `ruff format`: "X files left unchanged" or "X files reformatted"
- `ruff check`: "All checks passed!" or specific errors to fix

#### 3. Lint Check Only (CI validation)
```bash
python3 -m ruff check .
python3 -m ruff format . --check
```
**When to run**: To verify code passes CI checks without modifying files.

#### 4. Run Tests
```bash
pytest
# OR with coverage:
pytest --cov=custom_components.national_grid --cov-report=term-missing
```
**When to run**: After any code changes to verify functionality.

**Note**: Tests require `pytest-homeassistant-custom-component` which pulls in Home Assistant and many dependencies. Installation may take 2-3 minutes.

### Development Environment

#### Local Development with Home Assistant
```bash
scripts/develop
```
**What it does**:
1. Creates `config/` directory if it doesn't exist
2. Initializes Home Assistant configuration
3. Sets `PYTHONPATH` to include `custom_components/`
4. Starts Home Assistant on port 8123 in debug mode

**When to run**: To manually test the integration in a running Home Assistant instance.

**Requirements**: 
- Must have Python 3.13+
- Will create `config/` directory (gitignored except configuration.yaml)
- Home Assistant will be accessible at http://localhost:8123

#### DevContainer (VS Code)
The repository includes `.devcontainer.json` configured with:
- Python 3.13 container
- Port 8123 forwarded for Home Assistant
- Auto-runs `scripts/setup` on container creation
- Pre-configured VS Code extensions (ruff, python, pylance)

**To use**: Open in VS Code, click "Reopen in Container" when prompted.

## CI/CD Workflows

### Workflows Run on Every PR and Push to Main

#### 1. Lint Workflow (`.github/workflows/lint.yml`)
**Runs**: On push/PR to main
**Steps**:
1. Checkout code
2. Setup Python 3.13
3. Install requirements: `pip install -r requirements.txt`
4. Run `python3 -m ruff check .`
5. Run `python3 -m ruff format . --check`

**Failure causes**: 
- Formatting issues (run `scripts/lint` locally to fix)
- Linting errors (run `ruff check . --fix` or fix manually)

#### 2. Validate Workflow (`.github/workflows/validate.yml`)
**Runs**: On push/PR to main, daily at midnight, or manual dispatch
**Steps**:
1. **hassfest validation**: Home Assistant's official validator checks:
   - manifest.json structure and required fields
   - dependencies listed correctly
   - version format
   - Required files exist (strings.json, translations/, etc.)
2. **HACS validation**: Validates HACS compatibility:
   - hacs.json format
   - Repository structure
   - Integration category is "integration"
   - Currently ignores missing "brands" (brand images in home-assistant/brands repo)

**Failure causes**:
- Invalid manifest.json
- Missing required fields in hacs.json
- Incorrect integration structure
- Missing strings.json or translations

## Project Structure & Architecture

### Directory Layout
```
/
├── .github/
│   ├── workflows/          # CI workflows (lint.yml, validate.yml)
│   ├── ISSUE_TEMPLATE/     # GitHub issue templates
│   └── dependabot.yml      # Dependabot config (ignores homeassistant updates)
├── custom_components/
│   └── national_grid/      # Integration code (all Python files here)
│       ├── __init__.py     # Entry point, setup coordinator
│       ├── coordinator.py  # DataUpdateCoordinator, API calls
│       ├── config_flow.py  # UI configuration flow
│       ├── sensor.py       # Sensor entities (usage, cost)
│       ├── binary_sensor.py # Binary sensor entities (smart meter status)
│       ├── entity.py       # Base entity class
│       ├── const.py        # Constants (DOMAIN, units, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RyanMorash/ha_nationalgrid](https://github.com/RyanMorash/ha_nationalgrid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
