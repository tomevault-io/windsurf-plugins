---
trigger: always_on
description: This document provides comprehensive context for both LLM agents and human developers working on datui.
---

# Datui - Agent and Developer Documentation

This document provides comprehensive context for both LLM agents and human developers working on datui.

## Project Overview

Datui is a terminal user interface (TUI) for exploring and analyzing tabular data files. Built with Rust, Polars, and Ratatui, it provides an interactive environment for data exploration, querying, statistical analysis, and visualization within the terminal.

**Core Philosophy**: Memory-efficient data exploration using lazy evaluation, providing quick insights into datasets without requiring external tools or heavy infrastructure.

## Technology Stack

### Core Technologies

- **Rust**: Primary language for application core
- **Polars (0.52.0)**: Lazy API for all dataframe operations
  - Features: `lazy`, `parquet`, `json`, `polars-io`, `timezones`, `strings`, `pivot`
  - All operations use lazy evaluation by default
  - Data is collected only when needed for display
- **Ratatui (0.30.0)**: TUI framework for rendering
  - Uses `DefaultTerminal` with crossterm backend
  - Widget-based rendering system
- **Crossterm**: Terminal event handling and input

**NOTE: Agents are not permitted to downgrade package versions without express permission**

### Supporting Tools

- **Clap**: Command-line argument parsing
- **Color-eyre**: Error handling and reporting
- **Serde/Serde-json**: Template serialization (JSON format)
- **Dirs**: Standard cache/config directory management
- **FS2**: File locking for cache operations
- **Regex**: Pattern matching for queries

### Development Tools

- **Python + Polars**: Ad-hoc scripting and test data generation (`scripts/`)
- **Cargo**: Build system and testing
- **Rustfmt**: Code formatting (`cargo fmt`)
- **Clippy**: Linting and code quality checks (`cargo clippy`)

## Architecture Overview

### Application Structure

- **crates/datui-cli**: Shared CLI definitions (`Args`, `CompressionFormat`). Used by the main app, `build.rs` (manpage), and `gen_docs` binary (command-line-options markdown for mdbook).
- **src/bin/gen_docs.rs**: Emits `docs/reference/command-line-options.md` from Clap; run via `scripts/docs/generate_command_line_options.py` before mdbook.

```
src/
├── main.rs              # Entry point, CLI parsing, event loop
├── lib.rs               # App struct, event handling, main logic
├── widgets/             # UI widget implementations
│   ├── datatable.rs    # DataTable widget and state management
│   ├── controls.rs     # Control panel widget
│   ├── info.rs         # Info panel widget
│   ├── schema.rs       # Schema display widget
│   ├── debug.rs        # Debug overlay widget
│   ├── analysis.rs     # Analysis modal rendering
│   ├── chart.rs        # Chart view widget (sidebar + chart area)
│   ├── pivot_melt.rs   # Pivot & Melt modal rendering
│   └── template_modal.rs # Template management UI
├── query.rs            # Query parser and executor
├── chart_modal.rs      # Chart view state (type, x/y columns, options, export)
├── chart_data.rs       # Chart data preparation (LazyFrame → series points)
├── pivot_melt_modal.rs # Pivot & Melt modal state and specs
├── statistics.rs       # Statistical computation module
├── analysis_modal.rs   # Analysis modal state management
├── sort_filter_modal.rs # Sort & Filter tabbed modal state
├── sort_modal.rs       # Sort/column order (used by sort_filter)
├── filter_modal.rs     # Filter state (used by sort_filter)
├── template.rs         # Template storage and management
├── cache.rs            # Cache manager (query history, etc.)
└── config.rs           # Configuration manager (templates, etc.)
```

### Core Data Flow

1. **File Loading**: `main.rs` parses CLI args → `AppEvent::Open` → `App::event()` → `DataTableState::from_*()` → Creates `LazyFrame`
2. **User Input**: Terminal events → `AppEvent::Key` → `App::key()` → State mutation → `AppEvent` emission → Re-render
3. **Query Processing**: User types query → `query::parse_query()` → Polars expressions → `LazyFrame` transformations
4. **Display**: `App::render()` → Ratatui widgets → `terminal.draw()` → Terminal output

### Event System

**AppEvent Enum**: Central event type for all application actions
- `Key(KeyEvent)`: Keyboard input
- `Open(PathBuf, OpenOptions)`: File loading
- `Search(String)`: Query execution
- `Filter(Vec<FilterStatement>)`: Filter application
- `Sort(Vec<String>, bool)`: Sort application
- `ColumnOrder(Vec<String>, usize)`: Column reordering/locking
- `Pivot(PivotSpec)`: Pivot (long → wide) reshape
- `Melt(MeltSpec)`: Melt (wide → long) reshape
- `Collect`, `Update`, `Reset`: State management events
- `Exit`, `Crash(String)`: Lifecycle events

**Event Loop**: `main.rs` uses `mpsc::channel` for bidirectional communication
- Main thread: Polls terminal events, sends to channel
- App thread: Receives events, processes, may emit new events

### Configuration System

**Architecture**: 3-layer configuration with TOML format

**Layers** (priority order):
1. **Default Values**: Hardcoded in `Default` trait implementations
2. **User Config**: `~/.config/datui/config.toml` (XDG compliant)
3. **CLI Arguments**: Command-line flags (highest priority)

**Key Components**:
- `AppConfig`: Main configuration struct with all settings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [derekwisong/datui](https://github.com/derekwisong/datui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
