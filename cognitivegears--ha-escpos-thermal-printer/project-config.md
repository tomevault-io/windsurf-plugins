---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Home Assistant custom integration for ESC/POS thermal receipt printers. Supports **network (TCP/IP)**, **USB**, **Bluetooth**, and **serial (UART/RS-232)** connected printers. Enables printing text, QR codes, barcodes, images, and control commands (feed, cut, beep) through HA services and automations.

## Common Commands

```bash
# Install dependencies (use uv). Dev/test deps live in
# [project.optional-dependencies], so --all-extras pulls them in;
# there is no PEP 735 [dependency-groups] table (see pyproject.toml).
uv sync --all-extras

# Run tests (excludes integration tests by default)
uv run pytest -q

# Run a single test file
uv run pytest tests/test_services_text.py -v

# Run integration tests specifically
uv run pytest -m integration

# Linting and type checking (mypy needs an explicit target, as CI uses)
uv run ruff check .
uv run mypy custom_components/

# Pre-commit (runs automatically on commit)
pre-commit run --all-files

# Check requirements sync between pyproject.toml and manifest.json
python scripts/check_requirements_sync.py

# Auto-fix manifest.json from pyproject.toml
python scripts/sync_manifest_requirements.py

# Regenerate strings.json/translations/en.json 'services' key from services.yaml (add --check to only detect drift)
python scripts/sync_service_translations.py

# Run local Home Assistant with integration mounted (http://localhost:8123)
docker compose up -d
docker compose down  # to stop
```

## Architecture

### Testing (`tests/`)

- Unit tests use `pytest-homeassistant-custom-component` with async mode auto-enabled.
- Integration tests in `tests/integration_tests/` include a virtual printer emulator, mock data generators, and scenario tests.
- Tests marked `@pytest.mark.integration` require HA runtime and are excluded by default.
- `ESC_POS_DISABLE_PLATFORMS=1` is honored by `async_setup_entry`, but an autouse fixture in `tests/conftest.py` pins it to `0` for non-integration tests, so unit tests always run with full platform forwarding.

### Dependency Management

- **pyproject.toml** is source of truth for dependencies.
- **manifest.json** must mirror runtime deps (synced via `scripts/sync_manifest_requirements.py`).
- Dependabot auto-updates pyproject.toml (see `.github/dependabot.yml`); a post-upgrade task syncs manifest.json.
- Pre-commit hooks block commits if files drift.
- **Always use pinned versions** (`==`) for all dependencies, not ranges (`>=`). This ensures reproducible builds and better security.
- **`pytest`, `mypy`, `dbus-fast`, `Pillow`, `respx`, and `serialx` are constrained by HA core or the test harness** — before bumping any of them, upgrading `pytest-homeassistant-custom-component`, or triaging Dependabot security alerts, load the `dependency-pins` skill (`.claude/skills/dependency-pins/SKILL.md`) for the per-package rules.

## Key Patterns

- All printer I/O runs on executor threads via `hass.async_add_executor_job()`.
- Printer adapters use an `asyncio.Lock` to serialize print operations.
- Late import of `escpos.printer.Network` and `escpos.printer.Usb` avoids import errors during HA startup.
- Security validation happens before any printer operation (see `security.py` — single source of truth for `MAX_*` bounds, log sanitisation, and `O_NOFOLLOW` file primitives).
- `image_sources.py` builds a per-request `aiohttp` session pinned via `_StaticResolver` so DNS rebinding cannot swap public → private between validation and fetch.
- **Network printers:** Status checking uses non-blocking TCP probes.
- **USB printers:** Status checking uses USB device enumeration via `usb.core.find()`. Keepalive is always disabled (reconnect-per-operation model).
- USB printers are auto-discovered by matching vendor IDs in `THERMAL_PRINTER_VIDS`.
- Factory pattern (`create_printer_adapter()`) instantiates the correct adapter based on connection type.
- Unique IDs: Network uses `host:port`, USB uses `usb:VID:PID[:serial]`.

### Image services: field-set parity invariant

All six image-printing services (`print_image`, `print_image_url`, `print_image_path`, `print_camera_snapshot`, `print_image_entity`, `preview_image`) share a single voluptuous option-set mixin (`_image_option_fragment()` in `services/schemas.py`) and a single backend dispatcher (`_dispatch_print_image()` in `services/print_handlers.py`). Their `services.yaml` field definitions are therefore duplicated metadata — when adding/renaming/removing an option, update all six blocks in lockstep.

`tests/test_services_yaml_schema.py::test_image_services_share_common_field_metadata` enforces the invariant: every common field's `name`, `description`, and `selector` must match `print_image`'s. The `default:` *may* legitimately differ on `auto_resize`, `autocontrast`, and `feed` (each focused service picks its own friendly default) — those keys are listed in `_DEFAULT_MAY_VARY` in the test. Any drift outside that allowlist is a test failure.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cognitivegears/ha-escpos-thermal-printer](https://github.com/cognitivegears/ha-escpos-thermal-printer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
