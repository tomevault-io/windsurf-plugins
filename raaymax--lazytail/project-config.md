---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
cargo build                       # Debug build
cargo build --release             # Optimized release build
cargo test                        # Run fast tests only
cargo test -- --include-ignored   # Run all tests including slow ones
cargo test -- --ignored           # Run only slow/integration tests
cargo clippy                      # Lint checks
cargo fmt                         # Format code
cargo fmt -- --check              # Check formatting
cargo run -- test.log             # Run with a log file
```

Tests marked with `#[ignore]` are slow integration tests involving file system operations.

## Architecture

LazyTail is a terminal-based log viewer written in Rust using ratatui for the TUI.

### Event-Driven Core Loop (main.rs)

The application follows a render-collect-process cycle:
1. Render current state with ratatui
2. Check pending debounced filters, refresh source status, check directory watcher
3. Collect events from: file watcher, filter threads, user input
4. Process events via `App::apply_event()` to update state (all side-effects handled there)
5. Repeat until quit

`main.rs` (~1200 lines) handles terminal setup, event collection, and delegates all state changes to `App::apply_event()`. The `process_event()` function is a single-line passthrough.

### State Structure

- **App** (app/mod.rs): Top-level state decomposed into sub-controllers: `TabManager` (tabs/combined views), `InputController` (mode/buffer/cursor), `FilterController` (validation/debouncing/history), `SourcePanelController` (source tree), plus preset registry and theme
- **TabState** (app/tab.rs): Per-tab state including source, watcher, viewport, expansion
- **LogSource** (log_source.rs): Domain state for a log source — reader, index, filter config, line indices, rate tracker, aggregation result, renderer names, view modes (raw/wrap/timestamps). Shared across TUI/Web/MCP adapters.
- **Viewport** (app/viewport.rs): Vim-style scrolling with selection anchor and scrolloff padding

### Key Modules

- **app/**: Sub-modules: `event.rs` (AppEvent enum), `filter_controller.rs` (FilterController — debounce, validation, history), `input_controller.rs` (InputController, InputMode), `source_panel.rs` (SourcePanelController — tree navigation), `tab_manager.rs` (TabManager — tab collection, combined views, active tab)
- **reader/**: `LogReader` trait (4 methods: `total_lines`, `get_line`, `reload`, `as_any`) with `FileReader` (lazy O(1) line access via sparse index, lossy UTF-8 for binary tolerance), `StreamReader` (stdin buffering), and `CombinedReader` (multi-source chronological merging via index timestamps). `StreamableReader` trait extends `LogReader` with stream-specific methods (`append_lines`, `mark_complete`, `is_loading`) — only `StreamReader` implements it. This follows ISP: `FileReader` only implements `LogReader`.
- **filter/**: `Filter` trait with `StringFilter` and `RegexFilter`. `FilterEngine` runs filtering in background thread, sends progress via channel. `streaming_filter` provides mmap-based grep-like performance. `query/` directory implements field-based filtering (JSON/logfmt with text parser for `json | level == "error"` syntax) — `ast.rs` (FilterQuery AST), `parser.rs` (text parser), `filter.rs` (QueryFilter), `time.rs` (@ts time-based filtering). `search_engine` provides `SearchEngine` — stateless unified search dispatch (picks fastest execution path based on filter type, index, and range). `aggregation` provides grouped query results (`count by (field)` with `top N`).
- **filter_orchestrator.rs**: `FilterOrchestrator` — the unified entry point for all filter trigger paths (top-level module, not inside `filter/`)
- **handlers/**: Input, filter progress, and file event handlers
- **tui/**: ratatui rendering — `log_view.rs` (main log content), `side_panel.rs` (source tree), `status_bar.rs`, `help.rs` (keyboard shortcut overlay), `aggregation_view.rs` (grouped query results)
- **watcher/**: File watching (`file.rs` via notify/inotify) and directory watching (`dir.rs` for dynamic source discovery)
- **source.rs**: Source discovery, marker files (PID-based active/ended tracking), stale marker cleanup
- **log_source.rs**: `LogSource` struct (domain state shared across TUI/Web/MCP), `FilterConfig`, `LineRateTracker` (sliding-window ingestion rate)
- **capture.rs**: Capture mode (`-n` flag) — tee-like stdin-to-file with signal handling and incremental indexing
- **config/**: Config system — `discovery.rs` (lazytail.yaml walk parent dirs), `loader.rs` (YAML loading), `types.rs` (config structs), `error.rs`. Project-scoped vs global
- **cli/**: CLI subcommand definitions — `init.rs`, `config.rs`, `bench.rs` (filter performance benchmarking), `theme.rs` (theme import/list), `update.rs` (feature-gated: self-update)
- **renderer/**: Rendering preset system for structured log lines — `preset.rs` (compiled presets), `detect.rs` (auto-detection), `field.rs` (field extraction), `format.rs` (segment formatting), `segment.rs` (styled segments), `builtin.rs` (built-in presets)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raaymax/lazytail](https://github.com/raaymax/lazytail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
