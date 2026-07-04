---
trigger: always_on
description: This file documents conventions and operational guidance for automated agents that
---

AGENTS.md

Purpose
-------
This file documents conventions and operational guidance for automated agents that
work on this repository (CI bots, interactive coding agents, local dev automation,
etc.). Treat it as authoritative for style, build, and execution steps in the
worktree rooted at the repository root.

Use carefully: follow the repository-level rules and double-check any change that
mutates the codebase (commits, formatting, dependency changes).

Build / Lint / Test Commands
---------------------------
General (workspace) commands:
- Build workspace: `cargo build --workspace`
- Run all tests: `cargo test --workspace`
- Format all code: `cargo fmt --all`
- Lint (clippy) all crates: `cargo clippy --workspace --all-features -- -D warnings`

Build / run specific crates:
- Build a single crate: `cargo build -p <crate_name>`
- Run a specific binary: `cargo run -p <crate_name> --bin <binary_name>`
  Example GUI: `cargo run -p zaroxi-interface-desktop --bin gui_shell`

Feature-gated renderer (heavy GPU deps):
- Build with renderer features: `cargo build -p zaroxi-core-engine-render --features full_renderer`
- Run GUI with renderer feature: `cargo run -p zaroxi-interface-desktop --bin gui_shell --features full_renderer`

Running a single test
- From workspace root, run a single test in a crate:
  `cargo test -p <crate_name> <test_name>`
  Example: `cargo test -p zaroxi-core-engine-render test_rasterization`
- To see test output unbuffered (useful for println/eprintln in tests):
  `cargo test -p <crate_name> <test_name> -- --nocapture`
- For a test function using filters: `cargo test -p <crate_name> test_substring`

Running the UI in transcript/headless mode (CI-friendly):
- Some platform libs may not be available in CI. You can run the GUI in transcript
  mode which prints layout and logs instead of opening a window:
  `RUST_LOG=info cargo run -p zaroxi-interface-desktop --bin gui_shell`
- To enable text shader mask debug (diagnostics):
  `ZAROXI_TEXT_SHOW_MASK=1 RUST_LOG=info cargo run -p zaroxi-interface-desktop --bin gui_shell`

Code Style Guidelines
---------------------
Formatting and imports
- Always run `cargo fmt --all` before committing. The repository follows the
  Rust standard formatting. Agents should auto-format any modified files.
- Keep `use` imports grouped by crate (external crates first, workspace crates next
  or consistent with existing module ordering) and alphabetized where reasonable.
- Prefer explicit imports (`use crate::module::Type;`) over glob imports (`*`),
  except in test helpers or where the module provides a pre-defined re-export.

Types and naming
- Use full, descriptive names for variables and functions. Avoid single-letter
  variable names except in tight scopes like iterators (`i`, `j`) or short lambdas.
- Rust naming conventions:
  - Types, structs, enums, traits: `CamelCase` (eg. `InstanceSample`).
  - Functions, local variables: `snake_case` (eg. `prepare_buffer`).
  - Constants and static values: `UPPER_SNAKE_CASE` (eg. `DEFAULT_WIDTH`).
- Avoid Hungarian-style prefixes or cryptic abbreviations.
- Keep public API names stable and backwards-compatible where possible.

Error handling
- Use `Result<T, E>` for fallible functions. Prefer domain-specific error types
  (see `crate::error::RenderError`) over bare `anyhow` in library code.
- Bubble errors up with meaningful context using `thiserror`/`map_err`/`?`.
- Avoid silent `unwrap()` and `expect()` in library/core code. Tests and quick
  prototypes may use `unwrap()` but document it with a comment.
- When catching panics or performing defensive checks, log with `log::warn!` or
  `log::error!` with actionable messages.

Floating vs integer coordinates
- Keep layout math in floating point (`f32`) until the last possible step
  before rasterization or hardware upload. Avoid truncating to `i32` early; the
  cosmic/glyphon pipeline uses subpixel layout and metrics that must be preserved.

Concurrency and locking
- Use `Arc<Mutex<T>>` only where required by shared mutable state between threads.
- Minimize lock holdings; extract owned data from a borrow before calling into
  other components to avoid double mutable borrows.
- Prefer fine-grained locks for frequently-updated items; annotate why a particular
  mutex is required.

Logging
- Use `log::info!`, `log::debug!`, `log::warn!`, `log::error!` as appropriate.
- For diagnostic logs used by agents, prefer clearly prefixed lines such as
  `GUI_TEXT_GLYPH_RASTER:` or `GUI_TEXT_ATLAS_UPLOAD:` so tooling can easily
  grep them. Do not remove those diagnostic lines unless they are replaced by
  an improved diagnostic.

Tests
- Unit tests: keep them in the same module under `#[cfg(test)] mod tests { }`.
- Integration tests: place in `tests/` with descriptive names.
- When adding new behavior, include tests that exercise the behavior at the
  smallest scope that makes sense and include an integration test if it
  interacts across crates.

Documentation and comments
- Public items must have doc comments (`///`). Keep comments concise and
  explain why (not just what) for non-trivial logic.
- For patches that touch rendering or layout, add small inline comments explaining
  coordinate system choices (eg. physical vs logical pixels, baseline rules).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZaroxiHQ/zaroxi](https://github.com/ZaroxiHQ/zaroxi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
