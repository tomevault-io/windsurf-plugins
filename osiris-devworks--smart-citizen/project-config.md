---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Smart Citizen (formerly SC Localization Editor) is a Windows-only PyQt6 GUI application for customizing Star Citizen localization strings. Tagline: *Smarter Strings for Star Citizen*. Users configure multiple data sources (Global, Contracts, Components, Ships, Commodities, Gear, User) with a drag-and-drop merge hierarchy, edit strings in a table, and apply changes to their game installation with automatic backup management.

**Rebrand status**: As of 0.9.0, user-facing strings, registry path (`Osiris DevWorks\Smart Citizen`), and the user data root (`Documents\Smart Citizen\`) all use the new name. `AppSettings` still contains one-shot migrators for the legacy `Osiris DevWorks\SC Localization Editor` registry tree and `Documents\SC Localization Editor\` directory; do not remove them while users on pre-0.9 builds may still upgrade.

**Current Version**: Read from `VERSION.TXT` (single source of truth). Project is at 1.0 as of this writing.

## Quick Commands

```bash
# Setup (production deps only)
pip install -r requirements.txt

# Setup (with dev/test tools: pytest, flake8, black, mypy, etc.)
pip install -r requirements-dev.txt

# Run
python src/main.py

# Testing
pytest tests/                                    # Run all tests
pytest tests/test_core.py                       # Run single file
pytest tests/test_core.py::TestIniParsing       # Run single class
pytest tests/test_core.py::TestIniParsing::test_parse_basic_ini  # Run single test
pytest tests/ -v                                # Verbose output
pytest tests/ --cov=src --cov-report=html      # Coverage report (HTML)
pytest tests/ -n auto                           # Parallel execution (pytest-xdist)

# Code Quality
black src/ tests/ scripts/                      # Format code
flake8 src/ tests/ scripts/                     # Lint (use flake8 config if present)
isort src/ tests/ scripts/                      # Sort imports
mypy src/                                       # Type checking

# Building
cd scripts/build && python build_exe.py         # Build exe (PyInstaller)
cd scripts/build && build_all.bat               # Build exe + installer (requires Inno Setup)

# Data Generation
python scripts/generate_enhancements_ini.py [base_ini_path [dataforge_cache_dir]]
python scripts/extract_components.py [--stock path] [--base path] [--output path] [--dry-run]
```

## Testing Strategy

**Unit Tests** (`tests/`): Split by domain — `test_core.py` (INI parsing/merging/category extraction), `test_missions.py` (mission rewards pipeline), `test_pak_extraction.py` (P4K/DataForge), `test_progress_sink.py` (thread-safe progress coalescing), `test_dataforge_patcher.py` (declarative XML patching), `test_app_updater.py` (GitHub Releases version-check worker), `test_channel_layout.py` (per-channel directory migration). Pytest config lives in `tests/pytest.ini` (sets `pythonpath = src`, registers markers: `unit`, `integration`, `slow`, `critical`, `regression`).

**GUI Testing**: Manual. Run app (`python src/main.py`), load base file, edit a value, apply to game, restart to verify persistence. Use the Log Tab to watch for errors during load/merge/apply cycles.

## Architecture

Entry point: `src/main.py`. The app has two main layers:

**GUI layer** (`src/gui/`):
- `main_window.py` — Main window with table, toolbar, filters, backup/restore, threading workers, DataForge extraction. This is the largest file (~2000+ lines). Manages the primary workflow: load, merge, edit, apply.
- `config_tab.py` — **Config Tab**: Data source management (add/edit/remove sources), drag-drop merge hierarchy, Star Citizen install path, and DataForge extraction trigger.
- `enhancements_tab.py` — **Enhancements Tab**: Toggle stats overlays, configure ship favorites prefix, trigger DataForge extraction. Emits `merge_requested` and `stats_pipeline_requested` signals.
- `log_tab.py` — **Log Tab**: In-app real-time log viewer. Bridges Python `logging` to Qt text widget via `_LogEmitter` signal (thread-safe). Supports level filtering, auto-scroll, and log export.
- `filter_header.py` — `FilterHeaderView` QHeaderView subclass adding per-column QLineEdit filter row below header labels, with debounced filtering.
- `string_table_model.py` — `QAbstractTableModel` backing the strings `QTableView`. Replaces the old `QTableWidget.populate_table()` approach; renders visible rows on demand and sorts in Python (via `sort()` override) rather than per-comparison `lessThan()`. Column index constants (`COL_CATEGORY`, `COL_KEY`, `COL_DEFAULT`, `COL_CURRENT`, `COL_STAR`, `COL_CUSTOM`, `COL_STATUS`) live here.
- `import_dialog.py` — `ImportConflictDialog` for resolving conflicts when importing INI files into user overrides. Allows per-key resolution strategies (keep current, use imported, append, prepend, or custom).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Osiris-DevWorks/smart-citizen](https://github.com/Osiris-DevWorks/smart-citizen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
