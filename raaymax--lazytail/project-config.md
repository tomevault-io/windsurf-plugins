---
trigger: always_on
description: **Analysis Date:** 2026-02-03
---

# Coding Conventions

**Analysis Date:** 2026-02-03

## Naming Patterns

**Files:**
- Lowercase with underscores: `main.rs`, `filter_engine.rs`, `string_filter.rs`
- Module directories mirror file structure: `src/filter/mod.rs` exports submodules
- Trait implementations in separate files: `src/filter/string_filter.rs`, `src/filter/regex_filter.rs`

**Functions:**
- Snake_case for all functions: `run_app()`, `collect_file_events()`, `resolve_with_options()`
- Helper functions prefixed with verb: `handle_`, `process_`, `trigger_`, `collect_`
- Private helper functions explicitly documented with triple-slash comments
- Accessor methods use get_ prefix: `get_line()`, `get_screen_offset()`, `get_input()`
- Boolean checks use is_ or has_ prefix: `is_match()`, `is_loading()`, `is_case_sensitive()`

**Variables:**
- Snake_case for all variables: `active_tab`, `line_indices`, `scroll_position`, `filter_mode`
- Mutable collections suffix with _mut: Not observed; uses `mut` keyword instead
- Configuration constants in UPPER_SNAKE_CASE at module top: `const MAX_HISTORY_ENTRIES: usize = 50;`, `const DEFAULT_EDGE_PADDING: usize = 3;`
- Field names descriptive but concise: `anchor_line`, `scroll_position`, `edge_padding`, `input_buffer`

**Types:**
- PascalCase for all structs: `App`, `TabState`, `FileWatcher`, `StringFilter`, `RegexFilter`
- PascalCase for enums: `FilterMode`, `InputMode`, `ViewMode`, `FileEvent`, `AppEvent`
- Enum variants PascalCase: `FilterMode::Plain`, `InputMode::EnteringFilter`, `FileEvent::Modified`
- Type aliases match struct style: Not extensively used

## Code Style

**Formatting:**
- Standard Rust style via `cargo fmt`
- 4-space indentation (enforced by rustfmt)
- Maximum line length not explicitly enforced but generally ~80-100 chars
- Blank lines separate logical sections within functions
- Comments on own line for clarity

**Linting:**
- `cargo clippy` for lint checking
- Dead code marked with `#[allow(dead_code)]` with explanatory comment:
  ```rust
  #[allow(dead_code)] // Public API for external use and tests
  pub fn plain() -> Self {
  ```
- Unused imports removed, but `use` statements organized in groups
- Compiler warnings treated seriously (few in codebase)

## Import Organization

**Order:**
1. Standard library imports: `use std::path::PathBuf;`
2. External crate imports: `use ratatui::{...}; use regex::Regex;`
3. Internal crate imports: `use crate::filter::{...}; use crate::app::App;`
4. Conditional imports: `#[cfg(test)] use std::path::PathBuf;`

**Path Aliases:**
- Not heavily used; explicit full paths preferred
- Some convenience re-exports in mod.rs: `pub use self::engine::FilterEngine;`
- Glob imports avoided except in tests: `use super::*;`

## Error Handling

**Patterns:**
- Uniform error type: `anyhow::Result<T>` throughout codebase
- Error context added at call site: `.context("Failed to reload file")?`
- Internal errors propagated via `?` operator rather than `.unwrap()`
- Critical invariants use `assert!()` macro: `assert!(!tabs.is_empty(), "App must be created with at least one tab")`
- Graceful degradation: Invalid filters logged but don't crash, reader lock poisoning handled with explicit error message:
  ```rust
  reader_guard.reload()
      .expect("Reader lock poisoned - filter thread panicked")
  ```
- File operation errors include path in message for debugging
- Match on `Ok(x)` for expected success paths, ignore errors with `let _ = tx.send()`

**Error Recovery:**
- Errors in filter threads result in `FilterError` events, not panics
- File watcher errors logged to stderr but app continues: `FileEvent::Error(String)`
- Stream read errors marked as complete rather than crashing: `tab.mark_stream_complete()`

## Logging

**Framework:** Standard `println!()` and `eprintln!()` macros

**Patterns:**
- Debug info typically sent to stdout during normal operation
- Errors sent to stderr: `eprintln!("Failed to reload file for tab {}: {}", tab_idx, e);`
- Non-critical failures logged with context: `eprintln!("Warning: Failed to parse filter history: {}", e);`
- File modification tracking uses comments rather than logs: important state changes documented in code
- Tab index included in multi-tab operations for debugging:
  ```rust
  eprintln!("Filter error for tab {}: {}", tab_idx, err);
  ```

## Comments

**When to Comment:**
- Algorithm rationale: "During filtering, we used to force view to end. But this causes jumping when user navigates..."
- Non-obvious logic: Explain why something is done, not what it does
- SAFETY blocks for unsafe code: Required for all unsafe { } blocks
- Public APIs documented with `///` comments
- Complex state transitions explained: "First pass: reload files and handle inactive tabs"
- Configuration constants documented: "Debounce delay for live filter preview (milliseconds)"

**JSDoc/TSDoc:**
- Uses `///` doc comments for public items:
  ```rust
  /// Create a new viewport anchored to the given line
  pub fn new(initial_line: usize) -> Self {
  ```
- Parameter documentation: `/// The file line number that is selected (stable across filter changes)`
- Module-level documentation with `//!`: See `viewport.rs` and `source.rs`
- Examples in doc comments: Not extensively used
- Private function documentation sparse but present for complex logic

**Special Comments:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raaymax/lazytail](https://github.com/raaymax/lazytail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
