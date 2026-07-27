---
trigger: always_on
description: This repository contains **Shellies Discovery Gen2**, a Home Assistant `python_script`
---

# Repository Instructions

## Project Overview

This repository contains **Shellies Discovery Gen2**, a Home Assistant `python_script`
that generates MQTT discovery configuration for Shelly Gen2, Gen3, Gen4, and BLU
devices.

The main script is:

- `python_scripts/shellies_discovery_gen2.py`

It is intended to run inside Home Assistant's `python_script` integration, not as a
normal Python package. Keep the implementation compatible with that sandbox:

- Do not add runtime imports to `shellies_discovery_gen2.py`; Home Assistant
  `python_script` does not allow Python imports.
- Use the injected `data`, `hass`, and `logger` objects as the script interface.
- Keep helper code in the same script file unless it is test-only code.
- Treat Home Assistant behavior as the source of truth:
  - Python Script documentation:
    [https://www.home-assistant.io/integrations/python_script/](https://www.home-assistant.io/integrations/python_script/)
  - MQTT discovery documentation:
    [https://www.home-assistant.io/integrations/mqtt/#mqtt-discovery](https://www.home-assistant.io/integrations/mqtt/#mqtt-discovery)

## Python Version

The script runs in the Python environment bundled with Home Assistant. Always write
code compatible with **Python 3.14**. Do not use features or syntax from newer
versions without verifying Home Assistant support. Avoid type annotations in runtime
code — the `python_script` sandbox evaluates the script directly without a type
checker.

## Repository Layout

- `python_scripts/` — Home Assistant python scripts. The discovery generator lives
  here.
- `tests/` — pytest test suite.
- `tests/fixtures/` — full-device JSON fixtures used by device snapshot tests.
- `tests/snapshots/` — Syrupy snapshot output for MQTT topics and payloads.
- `scripts/` — helper scripts and documentation for Shelly-side scripts.
- `ruff.toml` — Ruff lint and formatting configuration.
- `.github/workflows/ci.yml` — CI lint and test commands.

## Script Architecture

The script receives device data through the `data` object injected by Home Assistant.
Understanding this flow is essential before modifying the script:

1. **Input (`data`)** — a dict containing device status, config, and component
   information retrieved from the Shelly device's RPC API. Key fields include:
   - `id` — device identifier (MAC-based)
   - `model` — Shelly model string (e.g., `"SNSW-001P16EU"`)
   - `components` — list of component dicts, each describing a sensor, switch, etc.

2. **Processing** — the script iterates over components and maps each to one or more
   Home Assistant MQTT discovery payloads using model constants and component
   definitions.

3. **Output** — MQTT discovery messages published via `hass.services.call()` to the
   `mqtt.publish` service. Each message targets a unique discovery topic.

### Supported Component Types (Platforms)

Each component type maps to a Home Assistant platform. Current supported platforms:

`binary_sensor`, `button`, `climate`, `cover`, `device_trigger`, `event`, `light`,
`number`, `select`, `sensor`, `switch`, `text`, `update`

When adding a new platform type, follow the pattern of an existing similar platform
and add tests covering the new component's discovery payload.

### BLU Device Architecture

BLU devices (e.g., Shelly BLU Button, BLU DW) differ from Gen2/Gen3/Gen4 devices:

- They do **not** communicate via direct MQTT. They use Bluetooth Low Energy and
  rely on a **Shelly gateway device** to relay their data over MQTT.
- The script generates discovery for BLU devices through the gateway's topics, not
  directly from the BLU device.
- When adding BLU device support, identify the gateway model that relays the device
  and ensure the fixture reflects the gateway + BLU component relationship.

## Development Setup

Install test dependencies using `uv` (required — do not use plain `pip`):

```bash
uv pip install -r requirements-test.txt
```

`uv` ensures reproducible environments consistent with CI. If `uv` is not installed:

```bash
pip install uv
```

## Development Commands

Run the same checks as CI:

```bash
ruff check python_scripts tests
ruff format --check python_scripts tests
pytest tests --timeout=30 --cov --cov-branch --cov-report=xml --error-for-skips
```

For local fixes:

```bash
ruff check --fix python_scripts tests
ruff format python_scripts tests
```

Run tests for a single device model only:

```bash
pytest tests/test_main.py -k "<fixture_name>"
```

Update Syrupy snapshots after an intentional behavior change:

```bash
pytest tests --snapshot-update
```

Always review the snapshot diff after `--snapshot-update` before committing.

Pre-commit-style repository hygiene is handled with `prek` and the hooks in
`.pre-commit-config.yaml`.

## Coding Guidelines

- Ruff is the linter and formatter. Follow `ruff.toml`; do not introduce local
  formatting styles that fight Ruff.
- Keep generated MQTT discovery payloads compact.
- Use Home Assistant MQTT discovery abbreviations for payload keys whenever an
  abbreviation exists:
  [https://www.home-assistant.io/integrations/mqtt/#supported-abbreviations-in-mqtt-discovery-messages](https://www.home-assistant.io/integrations/mqtt/#supported-abbreviations-in-mqtt-discovery-messages)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bieniu/ha-shellies-discovery-gen2](https://github.com/bieniu/ha-shellies-discovery-gen2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
