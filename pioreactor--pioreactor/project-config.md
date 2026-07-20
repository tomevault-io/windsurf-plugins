---
trigger: always_on
description: This repository contains the **executable code for the Pioreactor project**. It has primary sub-directories:
---

# Agent Guidelines for the Pioreactor Mono-Repo

This repository contains the **executable code for the Pioreactor project**. It has primary sub-directories:

1. `core/pioreactor` — backend / worker code that runs jobs. Important files:
   - `core/pioreactor/background_jobs/base.py` is the super class for background jobs (like stirring, od_reading, automations, etc)
2. `core/pioreactor/web/` — Flask-based web API. Important files:
   - `core/pioreactor/web/api.py` handles the leader-only (and frontend) API. This is the main entry point most often. It sends requests to `unit_api.py` too.
   - `core/pioreactor/web/unit_api.py` is the pioreactor-specific API for controlling individual actions on a Pioreactor.
   - `core/pioreactor/web/tasks.py` lists the Huey (background) tasks spawned by the web APIs.
3. `frontend/src` — React-based web UI
4. `packaging/` - contains files used to build, install, and run Pioreactor outside the normal Python package runtime. Most files here are provisioning inputs: they seed databases, config directories, system services, and $DOT_PIOREACTOR. Specific important files:
  - `packaging/runtime-files/pioreactor.env`
  - `packaging/shared-assets/sql/create_tables.sql`
  - `packaging/shared-assets/pioreactor/config.example.ini`

Some files under `packaging/runtime-files/` are shared into both leader and worker Raspberry Pi images by CustoPiZer. Do not infer leader-only behavior from the generic Linux leader installer; check CustoPiZer sync paths and image-owned service files before deciding packaging scope.


---

## Key components

* **Background Jobs**
  Long-running tasks inherit from `BackgroundJob` in `core/pioreactor/background_jobs/base.py`. Examples include stirring control (`stirring.py`), optical density readings (`od_reading.py`), temperature automation, dosing automation.

* **Automations**
  Higher-level automation logic derives from `AutomationJob` in `core/pioreactor/automations/base.py`. Dosing, temperature, and LED automations are implemented under `core/pioreactor/automations/`.

* **Command-Line Interface**
  The `pio` CLI in `core/pioreactor/cli/pio.py` provides commands to run jobs, adjust settings, view logs, and update software. It checks for first-boot success and ensures the user is not running as root.

* **Configuration System**
  Configuration is loaded through `get_config()` in `core/pioreactor/config.py`, which merges global and unit-specific files and creates additional sections like `PWM_reverse`. A sample development config is provided in `config.dev.ini` with settings for PWM channels, stirring parameters, OD reading, MQTT broker, UI options, and more.

* **Hardware Utilities**
  `core/pioreactor/hardware.py` defines GPIO pin mappings and I2C addresses depending on hardware version. Modules in `core/pioreactor/utils/` implement PWM control, ADC/DAC access, temperature sensors, and network helpers.

* **Data Structures and Messaging**
  Typed message structures for MQTT communication—such as `ODReadings`, `DosingEvent`, and `CalibrationBase`—are defined in `core/pioreactor/structs.py`.

* **Version and Device Info**
  Software version and hardware detection logic reside in `core/pioreactor/version.py`, exposing `__version__` and helper functions like `get_hardware_version()`.

* **Plugin System**
  Additional functionality can be loaded via Python entry points or drop-in `.py` files under `~/.pioreactor/plugins`. Plugins are discovered and registered in `core/pioreactor/plugin_management/__init__.py`.

* **Web API and UI**
  The `core/pioreactor/web/` directory includes our APIs and generated static frontend output.

---

## Running the system

ALWAYS use the project virtualenv, `.venv`, for any Python, mypy, or pytest commands.
Ignore the system Python version for project work. Use `.venv/bin/python`; this project targets Python 3.13.

## Environment model

This repo's local development behavior depends heavily on environment variables. Many bugs that look like code regressions are actually the process reading from the wrong environment root or using the wrong interpreter.

Common variables you should assume are meaningful:

- `TESTING=1`
- `GLOBAL_CONFIG`
- `DOT_PIOREACTOR`
- `RUN_PIOREACTOR`
- `PLUGINS_DEV`
- `PIO_EXECUTABLE`
- `PIOS_EXECUTABLE`

These are likely defined in the local `.envrc` file.

Do not assume bare `python`, `pytest`, or `mypy` point at the correct interpreter. Prefer `.venv/bin/python`, `.venv/bin/pytest`, and `.venv/bin/mypy`.

`DOT_PIOREACTOR` is the effective data root for much of the application. When debugging filesystem, calibration, profile, plugin, backup/restore, or config issues, confirm which `DOT_PIOREACTOR` root the process is using before changing code.

**Startup order (recommended):**

0. Before starting anything, run `make dev-status` to see whether the Huey consumer, Flask API (4999), or frontend dev server (3000) are already up. Only launch what's listed under "Need to start".
1. Start the Huey consumer:

   ```bash
   make huey-dev
   ```
2. Start the web API (port **4999**):

   ```bash
   make web-dev
   ```
3. Start the React dev server (port **3000**):

   ```bash
   make frontend-dev
   ```
4. (Optional) Run Pioreactor jobs, e.g.:

   ```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pioreactor/pioreactor](https://github.com/Pioreactor/pioreactor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
