---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

This project uses two repositories:

1. **UltraLog** (main repo) - Contains all source code, build configuration, and tests
2. **UltraLog.wiki** (separate repo) - Contains GitHub wiki documentation

The wiki is managed as a separate Git repository (standard GitHub wiki setup). When updating documentation:

- Code documentation (README.md, CLAUDE.md) stays in the main repo
- User-facing wiki pages (User-Guide.md, Supported-ECU-Formats.md, etc.) go in the wiki repo
- The wiki repo is typically located adjacent to the main repo (e.g., `../UltraLog.wiki/`)

## Project Overview

UltraLog is a high-performance ECU (Engine Control Unit) log viewer written in pure Rust. It parses log files from automotive ECUs (Haltech, ECUMaster, RomRaider, Speeduino, rusEFI, AiM, Link, etc.) and displays channel data as interactive time-series graphs with support for computed/virtual channels derived from mathematical formulas.

## Build Commands

```bash
# Development build
cargo build

# Release build (optimized)
cargo build --release

# Run the application
cargo run --release

# Run the test parser CLI utility
cargo run --bin test_parser

# Run tests
cargo test

# Check formatting
cargo fmt --all -- --check

# Run clippy lints
cargo clippy -- -D warnings
```

## Architecture

### Source Structure

```text
src/
├── main.rs           # Application entry point
├── lib.rs            # Library exports and module declarations
├── app.rs            # Main application state and eframe::App impl
├── state.rs          # Core data types and constants
├── units.rs          # Unit preference types and conversions
├── normalize.rs      # Field name normalization system
├── computed.rs       # Computed channels data types and library
├── expression.rs     # Formula parsing and evaluation engine
├── updater.rs        # Auto-update functionality
├── analytics.rs      # Privacy-respecting analytics
├── adapters/
│   ├── mod.rs        # Adapter module exports
│   ├── types.rs      # OpenECU Alliance spec types (AdapterSpec, ProtocolSpec, etc.)
│   ├── api.rs        # API client for fetching specs from openecualliance.org
│   ├── cache.rs      # Local disk cache at {app_data_dir}/UltraLog/oecua_specs/
│   └── registry.rs   # Spec registry with fallback chain (cache -> embedded -> API)
├── parsers/
│   ├── mod.rs        # Parser module exports
│   ├── types.rs      # Core parser types (Log, Channel, Value, etc.)
│   ├── haltech.rs    # Haltech ECU log parser
│   ├── ecumaster.rs  # ECUMaster EMU Pro CSV parser
│   ├── romraider.rs  # RomRaider CSV parser
│   ├── speeduino.rs  # Speeduino/rusEFI MLG binary parser
│   ├── aim.rs        # AiM XRK/DRK binary parser
│   ├── link.rs       # Link ECU LLG binary parser
│   └── woolich.rs    # Woolich Racing Tuned CSV parser
└── ui/
    ├── mod.rs                        # UI module exports
    ├── sidebar.rs                    # File list and view options panel
    ├── channels.rs                   # Channel selection and cards
    ├── chart.rs                      # Chart rendering, legends, LTTB algorithm
    ├── timeline.rs                   # Timeline scrubber and playback controls
    ├── menu.rs                       # Menu bar (Units, Tools, Help menus)
    ├── toast.rs                      # Toast notification system
    ├── icons.rs                      # Custom icon drawing utilities
    ├── tab_bar.rs                    # Multi-file tab interface
    ├── tool_switcher.rs              # Switch between chart and scatter plot
    ├── scatter_plot.rs               # XY scatter plot visualization
    ├── export.rs                     # PNG and PDF export functionality
    ├── normalization_editor.rs       # Custom field mapping editor
    ├── computed_channels_manager.rs  # Computed channels library UI
    ├── formula_editor.rs             # Formula creation and editing
    └── update_dialog.rs              # Auto-update notification dialog
```

### Core Modules

- **`app.rs`** - Main `UltraLogApp` struct with application state. Contains:
  - File loading (background threads via `std::sync::mpsc`)
  - Channel management (add/remove/color assignment)
  - Cursor and time range tracking
  - eframe::App implementation

- **`state.rs`** - Core data types:
  - `LoadedFile` - Represents a parsed log file
  - `SelectedChannel` - A channel selected for visualization
  - `CacheKey`, `LoadResult`, `LoadingState` - Internal state types
  - Color palette constants (`CHART_COLORS`, `COLORBLIND_COLORS`)

- **`units.rs`** - Unit preference system:
  - Enums for each unit type (Temperature, Pressure, Speed, etc.)
  - `UnitPreferences` struct for storing user selections
  - Conversion methods between metric/imperial units

- **`normalize.rs`** - Field name normalization:
  - Maps ECU-specific channel names to standardized names
  - Built-in mappings for common channels across ECU systems
  - Custom mapping support via UI editor

- **`computed.rs`** - Computed/virtual channels:
  - `ComputedChannelTemplate` - Reusable formula templates with metadata
  - `ComputedChannel` - Instantiated channel with bindings and cached data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ClassicMiniDIY/UltraLog](https://github.com/ClassicMiniDIY/UltraLog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
