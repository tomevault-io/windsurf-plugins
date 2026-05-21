---
trigger: always_on
description: GUI application for batch converting videos to AV1 using VMAF-targeted quality encoding via the `ab-av1` tool.
---

# Auto-AV1-Converter

GUI application for batch converting videos to AV1 using VMAF-targeted quality encoding via the `ab-av1` tool.

## Tech Stack

- **Python 3** with Tkinter GUI
- **UV** for package management
- **Ruff** for linting/formatting
- **ty** for type checking
- **External tools**: `ab-av1`, FFmpeg with libsvtav1 (downloaded to `vendor/` or system PATH)

## Commands

```bash
# Run application
python -m src.convert          # or convert.bat (Windows)

# Development
uv sync                        # Install dev dependencies
uv run ruff check src/         # Lint
uv run ruff check --fix src/   # Lint with auto-fix
uv run ruff format src/        # Format
uv run ty check src/           # Type check
```

## Project Structure

```
src/
├── convert.py                 # Entry point
├── main.py                    # App initialization, Tkinter setup
├── config.py                  # Constants (VMAF targets, presets)
├── models.py                  # Dataclasses (ProgressEvent, ConversionConfig, FileRecord, QueueItem, OperationType, etc.)
├── estimation.py              # Time estimation from history
├── utils.py                   # Formatting helpers, ffprobe, UI thread safety
├── video_conversion.py        # Single-file conversion logic
├── folder_analysis.py         # Analysis tab: scanning, estimation, file classification
├── history_index.py           # Thread-safe O(1) cache for FileRecord lookups
├── cache_helpers.py           # CRF cache validation and reuse logic
├── logging_setup.py           # Logging configuration with rotating handlers
├── platform_utils.py          # Windows subprocess hiding, power management
├── privacy.py                 # Path anonymization (BLAKE2b hashing)
├── hardware_accel.py          # Hardware-accelerated decoding (CUVID, QSV)
├── video_metadata.py          # Video metadata extraction from ffprobe
├── vendor_manager.py          # ab-av1/FFmpeg download and update management
├── ab_av1/                    # ab-av1 wrapper package
│   ├── wrapper.py             # Subprocess management, VMAF fallback
│   ├── parser.py              # Regex parsing of ab-av1/ffmpeg output
│   ├── exceptions.py          # Custom exception hierarchy
│   ├── checker.py             # ab-av1 availability check
│   └── cleaner.py             # Temp folder cleanup
├── conversion_engine/         # Batch conversion (no GUI imports)
│   ├── worker.py              # Sequential worker thread
│   ├── scanner.py             # Video file scanning/filtering
│   └── cleanup.py             # Temp folder cleanup scheduling
└── gui/                       # Tkinter GUI
    ├── main_window.py         # Main window, settings persistence
    ├── base.py                # Base GUI components (explorer, tooltips)
    ├── constants.py           # Centralized UI colors, fonts, styling
    ├── conversion_controller.py # Start/stop/force-stop logic, callback dispatcher
    ├── analysis_controller.py # Analysis tab coordination/events
    ├── queue_controller.py    # Queue tab event handling
    ├── callback_handlers.py   # Event handlers (progress, completed, error, etc.)
    ├── gui_updates.py         # Thread-safe UI updates
    ├── gui_actions.py         # User interaction handlers
    ├── analysis_scanner.py    # Incremental folder scanning with ffprobe
    ├── analysis_tree.py       # Analysis tree display/state management
    ├── queue_manager.py       # Queue item creation/categorization
    ├── queue_tree.py          # Queue tree display/state management
    ├── tree_utils.py          # Tree expand/collapse utilities
    ├── tree_display.py        # Shared tree status formatting
    ├── tree_formatters.py     # Time/size/efficiency formatting and parsing
    ├── dependency_manager.py  # ab-av1/FFmpeg version checking and updates
    ├── charts.py              # Canvas-based chart drawing (bar, pie, line)
    ├── tabs/                  # Tab implementations
    │   ├── analysis_tab.py    # Analysis tab UI definition
    │   ├── convert_tab.py     # Convert tab with queue and progress
    │   ├── settings_tab.py    # Settings tab
    │   └── statistics_tab.py  # Statistics/history tab
    ├── dialogs/               # Modal dialog windows
    │   └── ffmpeg_download_dialog.py  # FFmpeg download confirmation
    └── widgets/               # Reusable UI components
        ├── operation_dropdown.py   # In-cell operation dropdown for queue
        └── add_to_queue_dialog.py  # Preview dialog for queue additions

tools/
└── hash_lookup.py             # Reverse lookup for anonymized file hashes
```

## Architecture

### Two-Phase Conversion

1. **Quality Detection**: ab-av1 samples video at various CRF values to find one meeting VMAF target
2. **Encoding**: FFmpeg encodes full video with optimal CRF

### VMAF Fallback

If target VMAF (default 95) is unattainable, decrements by 1 down to minimum (90), then skips as "not worthwhile".

### Threading Model

- **Main thread**: Tkinter event loop
- **Worker thread**: `sequential_conversion_worker()` handles conversion
- **Analysis threads**: `ThreadPoolExecutor` with 4-8 parallel ffprobe workers
- **GUI updates**: All UI changes via `utils.update_ui_safely()` → `root.after()`

### Analysis Tab (Four-Level Model)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Loufe/AB-AV1-GUI](https://github.com/Loufe/AB-AV1-GUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
