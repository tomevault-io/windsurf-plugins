---
trigger: always_on
description: EliteMining is a comprehensive Python/Tkinter application for Elite Dangerous mining automation and analytics, supporting both standalone and VoiceAttack-integrated modes. The codebase features robust configuration with rate limiting, advanced cargo monitoring with ship change detection, session tracking, and TTS announcements, with a focus on reliability and seamless Elite Dangerous integration.
---

# EliteMining AI Coding Agent Instructions

## Project Overview
EliteMining is a comprehensive Python/Tkinter application for Elite Dangerous mining automation and analytics, supporting both standalone and VoiceAttack-integrated modes. The codebase features robust configuration with rate limiting, advanced cargo monitoring with ship change detection, session tracking, and TTS announcements, with a focus on reliability and seamless Elite Dangerous integration.

**Core Architecture**: Multi-threaded Tkinter GUI with background Elite Dangerous journal monitoring, cargo tracking via multiple JSON sources (Journal, Cargo.json, Status.json), Windows SAPI TTS integration, and bidirectional VoiceAttack communication through NATO phonetic alphabet text files.

**Version**: 4.0.5 (see `app/version.py` for current version constants and config schema versioning)

## Key Components & Data Flows

### Core Application Architecture
- **Main GUI (`main.py`)**: Multi-tabbed Tkinter application with sophisticated **ToolTip system** (global enable/disable, positioning logic for edge cases), **CargoMonitor class** for real-time multi-source cargo tracking with background monitoring threads, and **dark theme styling** via ttk.Style configuration.
- **Session Management (`prospector_panel.py`)**: Elite Dangerous journal file monitoring with **startup skip logic** to prevent processing old events, deduplication via event tracking, and integration with `mining_statistics.py` for analytics.
- **TTS System (`announcer.py`)**: Windows SAPI integration with voice fallback logic, **diagnostic capabilities** for voice recycling issues, and cleanup of debug messages for production.
- **Version Management (`version.py`)**: Application versioning with separate config schema version tracking for backward compatibility (`__version__`, `__config_version__`, `__build_date__`).

### Configuration & State Management
- **Configuration (`config.py`, `config.json`)**: **Rate-limited config loading** (2-second cache) via `_load_cfg()`/`_save_cfg()`, **context-aware path detection** using `_get_config_path()` for development vs. compiled executable environments, and atomic file operations via `_atomic_write_text()`.
- **Ship Presets**: JSON-based ship configurations in `Settings/*.json` containing firegroups, timers, toggles with automatic binding to UI StringVar/IntVar for persistence.

### Elite Dangerous Integration
- **Multi-Source Cargo Monitoring**: CargoMonitor class tracks cargo via **Journal events, Cargo.json, and Status.json** with `refresh_ship_capacity()` for automatic ship change detection and **background monitoring threads** (`_start_background_monitoring()`) that work without UI windows.
- **Journal Processing**: Prospector event parsing with **material classification** (RARE_MATERIALS, COMMON_MATERIALS), announcement filtering, and session lifecycle management.
- **Real-Time Data Paths**: `~\Saved Games\Frontier Developments\Elite Dangerous\` contains Journal*.log, Cargo.json, Status.json files monitored continuously for game state changes.

### VoiceAttack Communication
- **NATO Phonetic Variables**: Firegroup mappings use NATO alphabet in `Variables/*.txt` files (A→"Alpha", C→"Charlie") with `NATO_REVERSE` dict for bidirectional conversion.
- **EliteVA Plugin**: Full Elite Dangerous API integration via `app/EliteVA/` with proper MIT license attribution and keybinding files.
- **Variable Structure**: `VA_VARS` dict maps tools to variable files: `{"Mining lasers": {"fg": "fgLasers", "btn": "btnLasers"}}` with atomic writes to prevent VoiceAttack reading partial files.

## Essential Files & Directories
- `app/main.py` (CargoMonitor class), `app/prospector_panel.py`, `app/announcer.py`, `app/mining_statistics.py`, `app/config.py`
- `config.json` (window geometry, TTS, announce map), `app/Settings/*.json` (ship presets)
- `Variables/*.txt` (VoiceAttack variables, NATO format)
- `app/EliteVA/` (EliteVA plugin with MIT license), `LICENSE.txt` (EliteVA licensing)
- `Configurator.spec`, `create_release.py`, `EliteMiningInstaller.iss`, `build_eliteMining_with_icon.bat`
- `app/Images/`, `app/Reports/`, `app/Output/`

## Development Patterns & Conventions

### Configuration Management Patterns
- **Rate-Limited Loading**: Always use `_load_cfg()`/`_save_cfg()` with 2-second cache to prevent config spam (20+ loads prevented per operation). Never bypass this system.
- **Context-Aware Paths**: Use `_get_config_path()` for automatic dev vs. compiled path detection. Development mode uses `app/config.json`, compiled uses parent `EliteMining/config.json`.
- **Atomic File Operations**: Use `_atomic_write_text()` for all VoiceAttack variable writes to prevent partial file reads during VA polling cycles.

### UI Architecture Patterns  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Viper-Dude/EliteMining](https://github.com/Viper-Dude/EliteMining) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
