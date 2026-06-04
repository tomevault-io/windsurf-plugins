---
trigger: always_on
description: This repository is a **custom integration for [Home Assistant](https://www.home-assistant.io/)**, written in Python.
---

# GitHub Copilot Custom Instructions for `hass-diagral`

## Project Context

- **Project Type:**  
  This repository is a **custom integration for [Home Assistant](https://www.home-assistant.io/)**, written in Python.
- **Purpose:**  
  The integration connects Home Assistant to the Diagral alarm system, exposing alarm states, sensors, and diagnostics, and providing Home Assistant entities, events, and configuration flows.
- **Structure:**  
  - All integration code is in `custom_components/diagral/`.
  - Documentation is in the `docs/` directory and `docs.json`.
  - The integration uses Home Assistant's config flow, entity platforms (`sensor`, `alarm_control_panel`), and `DataUpdateCoordinator` pattern.
  - The codebase follows Home Assistant's best practices for custom components.
- **Language:**  
  - All code, comments, and docstrings must be in **English** (even though the maintainer is French).
- **Documentation:**  
  - All files in `docs/` and `docs.json` are documentation and must use the `docs` commit type.
  - Documentation is in Markdown or MDX, and must be clear and concise.
  - All new entities must be documented in `docs/integration/entities.mdx`.
- **Testing:**  
  - Use `pytest` conventions for tests.
  - Tests live at `custom_components/diagral/tests/` (inside the devcontainer-mounted folder).
  - **Whenever any `custom_components/diagral/*.py` file is created or modified, immediately analyse the corresponding `tests/test_<module>.py` and CREATE, UPDATE, or DELETE tests as needed. Never skip this step silently.**
  - After writing or modifying tests, run the test suite (see Quick Start below for environment detection and the correct command).
- **Linting/Formatting:**  
  - Use `flake8` as the linter, with a maximum line length of 150 characters.
  - Run locally with: `flake8 --max-line-length=150 custom_components/diagral/`
  - You may also use `black` for formatting, but `flake8` is required for linting.

## Quick Start — Key Commands

```bash
# Lint
flake8 --max-line-length=150 custom_components/diagral/

# Tests — environment-aware (see tests.instructions.md for full details)
# 1. Check if inside devcontainer:
test -d /workspaces && echo "inside" || echo "outside"
# 2a. Inside devcontainer:
#   cd /workspaces/home-assistant-dev/config/custom_components/diagral && pytest tests/ -v
# 2b. Outside devcontainer — list containers, then:
#   docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Names}}"
#   docker exec -w /workspaces/home-assistant-dev/config/custom_components/diagral <ID> pytest tests/ -v

# CI validation (run by GitHub Actions — not local)
# hassfest  →  validates manifest.json, translations, quality_scale.yaml
# hacs/action  →  validates HACS compatibility
```

> **No Makefile** — there is no automated local build script. Use the commands above directly.
> A `pyproject.toml` lives at `custom_components/diagral/pyproject.toml` for pytest configuration (mounted into the devcontainer).

## Architecture

```
custom_components/diagral/
├── __init__.py          # Integration setup/teardown, webhook registration
├── coordinator.py       # DiagralDataUpdateCoordinator (polls every 300 s)
├── models.py            # DiagralData, DiagralConfigData, DiagralOptionsData
├── const.py             # All constants (DOMAIN, services, default values)
├── config_flow.py       # UI config flow + options flow
├── entity.py            # Base entity class (DiagralEntity)
├── alarm_control_panel.py  # AlarmControlPanel platform
├── sensor.py            # Sensor platform
├── webhook.py           # Webhook handler (push updates from Diagral cloud)
├── diagnostics.py       # HA diagnostics endpoint
├── services.yaml        # Custom service definitions
├── manifest.json        # Integration metadata (pydiagral==1.6.0)
└── translations/        # en.json, fr.json
```

**Key patterns:**
- `DiagralConfigEntry = ConfigEntry[DiagralData]` — typed config entry shorthand.
- `DiagralDataUpdateCoordinator` fetches: `alarm_config`, `devices_infos`, `groups`, `system_status`, `anomalies`.
- Webhook supports both **Nabu Casa cloud** (`.nabu.casa` domain) and direct HA webhook.
- All coordinator data comes from **`pydiagral`** (the external library wrapping the Diagral API).

**External dependency:** `pydiagral==1.6.0` (pinned). All Diagral API calls go through this library.

## CI/CD

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `home-assistant.yaml` | push / PR | `hassfest` + HACS validation |
| `release.yaml` | tag push | `semantic-release` (Node.js-based) |
| `codeql.yaml` | weekly + push | CodeQL security analysis (Python) |
| `devsec.yaml` | `dev` branch / PR to `dev` | FortiDevSec SAST scan |

**Branch strategy:** `dev` is the integration/development branch. `main` is stable (releases). All PRs must target `dev`.

## External Context and Libraries

- You may use context7 for code generation and suggestions.
- You are allowed to use libraries and APIs from:
  - `/home-assistant/core`
  - [developers.home-assistant.io](https://developers.home-assistant.io)
- Use these resources to ensure compatibility and best practices with Home Assistant integrations.

## Pull Request Best Practices

- **Title:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mguyard/hass-diagral](https://github.com/mguyard/hass-diagral) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
