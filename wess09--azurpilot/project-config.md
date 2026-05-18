---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AzurLaneAutoScript (ALAS / AzurPilot) is an automation framework for the mobile game Azur Lane. It controls Android emulators via ADB/uiautomator2, takes screenshots, recognizes UI elements through image matching and OCR, and executes game tasks automatically. Supports CN/EN/JP/TW game servers with server-specific assets. Licensed under GPL-3.0.

**Design constraints**: Designed for 7×24h continuous operation. Real Android devices are not supported (black screen/freeze under long runs, screenshot compression, OCR model migration issues). Fixed 1280×720 resolution only — best balance between clarity and screenshot latency, non-standard aspect ratios have no unified standard.

## Commands

This project uses **uv** for Python dependency management and script execution. All Python commands should be run via `uv run`.

### Environment Setup
```bash
uv venv                                        # Create virtual environment
uv pip install -r requirements-linux.txt       # Linux
uv pip install -r requirements.txt             # Windows
```

### Running the Application
```bash
uv run gui.py          # Start WebUI server (default port 22267)
uv run alas.py          # Run scheduler directly (headless)
```

### Linting
```bash
# Python (CI uses ruff with permissive settings - fatal syntax errors and undefined names only)
uv run ruff check . --select E9,F63,F7,F82 --ignore F821,F722

# Webapp (Electron/Vue)
cd webapp && pnpm lint
cd webapp && pnpm typecheck
```

### Building the Webapp
```bash
cd webapp && pnpm install
cd webapp && pnpm build && pnpm compile
```

### Config Generation (required after modifying config YAML files)
```bash
uv run -m module.config.config_updater
```
This regenerates `args.json`, `menu.json`, `config_generated.py`, `template.json`, and `i18n/*.json`.

### Asset Management
```bash
uv run -m dev_tools.button_extract    # Extract button definitions from screenshots
```

## Architecture

### Core Flow
1. `gui.py` starts the web server and manages config instances
2. `alas.py` (`AzurLaneAutoScript`) is the task runner — loads config, initializes device, dispatches tasks to handlers
3. Each task handler (e.g., `module/research/research.py`) inherits from base classes, uses the device for screenshots, UI detection, and input
4. `module/device/device.py` wraps ADB/uiautomator2 for screenshot capture and touch input
5. UI navigation (`module/ui/ui.py`) handles page detection and routing between game screens
6. Template matching (`module/base/template.py`) and OCR (`module/ocr/`) identify game UI elements

### Entry Points
- **`alas.py`** — Core scheduler. `AzurLaneAutoScript.loop()` runs an infinite scheduling loop: pick next task by priority, dispatch to method, handle errors, sleep until next task.
- **`gui.py`** — WebUI backend (PyWebIO + Starlette + uvicorn). Each ALAS config instance runs in its own `multiprocessing.Process`.
- **`mcp_server_sse.py`** — MCP server exposing 18 tools over SSE for external AI assistant integration.

### Module Layer Structure (`module/`)

**Base layer** (`module/base/`):
- `ModuleBase` (`base.py`) — Root class for all game logic. Composes `AzurLaneConfig` + `Device`. Provides `appear()`, `appear_then_click()`, `loop()`, image utilities.
- `Button` (`button.py`) — UI element with bounding box, color, click area, template image. Supports server-specific assets.
- `Template` (`template.py`) — Template matching against screenshots.
- `Resource` (`resource.py`) — Base class tracking all Button/Template instances, supports cache release.

**Device layer** (`module/device/`):
- `Device` (`device.py`) — Multiple inheritance: `Screenshot + Control + AppControl + Input`. Unified interface for emulator interaction.
- `module/device/method/` — Multiple screenshot/input backends: adb, minitouch, maatouch, droidcast, uiautomator2, nemu_ipc, ldopengl, hermit, wsa, ascreencap.
- `module/device/platform/` — Emulator management (LDPlayer, BlueStacks, NoxPlayer, MuMu, etc.).

**Config system** (`module/config/`):
- `config/template.json` defines the schema and defaults for all config options.
- `module/config/config_generated.py` is auto-generated from `template.json` — provides IDE autocomplete.
- `module/config/config_updater.py` regenerates `config_generated.py` when `template.json` changes.
- `module/config/config.py` (`AzurLaneConfig`) loads user config from `config/{config_name}.json` and merges with the template.
- User config files are stored in `config/{config_name}.json` (e.g., `config/alas.json`).
- 3-layer YAML pipeline for GUI: `task.yaml` (task→group mapping) → `argument.yaml` (group→argument definitions) → `override.yaml` (value/display patches).
- `config_updater.py` generates: `args.json`, `menu.json`, `config_generated.py`, `template.json`, `i18n/*.json`.
- Config path format: `<Task>.<Group>.<Argument>` (e.g., `Main.Campaign.Name`).
- Access config: `self.config.Group_Argument` (underscore-separated).

**UI navigation** (`module/ui/`):
- `Page` (`page.py`) — Graph-based navigation. Each page has a `check_button` and `links` dict. Uses A* pathfinding for shortest navigation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wess09/AzurPilot](https://github.com/wess09/AzurPilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
