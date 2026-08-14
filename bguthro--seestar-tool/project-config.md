---
trigger: always_on
description: Rust desktop application for managing firmware on ZWO Seestar smart telescopes.
---

# CLAUDE.md — Seestar Tool

Rust desktop application for managing firmware on ZWO Seestar smart telescopes.
Built with egui (GUI) and ratatui (TUI).

---

## Building

```bash
cargo build           # debug
cargo build --release # release (stripped, opt-level 3)
```

Linux requires system GUI libraries — see README for the `apt-get` list.

---

## Running checks

CI runs these in order; all must pass before merging:

```bash
cargo fmt -- --check   # formatting (enforced, not just style)
cargo clippy -- -D warnings  # all clippy warnings are errors
cargo build
cargo test
```

Run `cargo fmt` (no `--check`) to fix formatting in place.

---

## Lints

The following are **hard errors** (see `[lints.rust]` in Cargo.toml):

- `dead_code`
- `unused_imports`
- `unused_mut`
- `unused_parens`

`clippy::all` is set to warn, and CI promotes every warning to an error via `-D warnings`.
Do not suppress lints with `#[allow(...)]` unless there is a concrete, documented reason.

---

## Code style

- **No external test fixtures** — all test data is constructed in-memory with builder helpers.
- **No `unwrap()` in production code paths** — use `?` and `anyhow::Result`.
- **`unwrap()` is fine inside tests** — panics on assertion failure is the right behavior there.
- Public functions in each module are documented with `///` doc comments. Private helpers do not need docs unless the logic is non-obvious.
- Module-level doc comments (`//!`) describe the module's responsibility and call out the Python reference file it mirrors (if any).

---

## GUI / TUI feature parity

Every user-facing feature **must be implemented in both interfaces**. `gui.rs` (egui) and
`tui.rs` (ratatui) are parallel front-ends over the same backend logic in `runner.rs`.
A feature is not complete until it works in both.

This applies to:
- New operations (e.g., a new firmware source, a new scope action)
- New user-configurable inputs or options
- New status/progress feedback shown during a task
- New confirmation dialogs or warnings

When adding a feature, implement the GUI side and the TUI side in the same commit (or PR).
Do not merge a GUI-only or TUI-only implementation.

---

## Source layout

| File | Responsibility |
|---|---|
| [src/apk.rs](src/apk.rs) | APK/XAPK unpacking, AXML version parsing, PEM extraction from APKs |
| [src/firmware.rs](src/firmware.rs) | iscope extraction, validation, OTA upload, scope model detection |
| [src/apkpure.rs](src/apkpure.rs) | APKPure scraping and download |
| [src/pem.rs](src/pem.rs) | PEM key scanning and extraction from raw bytes |
| [src/task.rs](src/task.rs) | Shared message types (`TaskMsg`) for background task channels |
| [src/runner.rs](src/runner.rs) | Background task orchestration (firmware install, download, PEM extract) |
| [src/gui.rs](src/gui.rs) | egui front-end |
| [src/tui.rs](src/tui.rs) | ratatui terminal UI |
| [src/main.rs](src/main.rs) | Entry point, CLI flag parsing |

---

## Unit tests

### Requirements for all new features

Every new function with non-trivial logic **must** have unit tests. The bar is:

1. **Happy path** — the function returns the expected result on valid input.
2. **Error paths** — every early-return `Err(...)` branch must be exercised with a test that asserts on the error message or error kind.
3. **Boundary conditions** — off-by-one sizes, empty inputs, minimum/maximum values.

Tests that are not yet written are not optional; do not submit a feature without them.

### Where tests live

Tests go at the **bottom of the same source file** as the code they test, inside:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    // ...
}
```

Do not create separate test files. This keeps the test helpers close to the code they exercise and avoids a proliferating `tests/` directory.

### Naming convention

Test functions are named `<function_being_tested>_<what_is_asserted>`:

```rust
#[test]
fn validate_iscope_rejects_empty_data() { ... }

#[test]
fn is_xapk_false_for_plain_apk() { ... }

#[test]
fn open_apk_returns_error_for_missing_iscope_entry() { ... }
```

Snake case throughout. No `test_` prefix — the `#[test]` attribute is sufficient.

### Building test data in-memory

Tests **never** load files from disk except through a `TempFile` RAII helper (see below).
Instead, construct the minimal data structure needed for each case using a local builder
helper. Examples already in the codebase:

- `make_zip(&[("path", bytes)])` → in-memory ZIP
- `make_fake_iscope(elf_class)` → real tar.bz2 with a minimal fake ELF
- `build_axml(version)` → binary AndroidManifest.xml chunk

Write a new builder helper if the existing ones do not cover the format you need. Keep helpers
in the same `mod tests` block, not at the top level.

### TempFile for disk-required paths

When the function under test requires a filesystem path (e.g., `open_apk` takes `&str`),
use the `TempFile` RAII helper that already exists in each module's test block:

```rust
struct TempFile(std::path::PathBuf);
impl TempFile {
    fn write(name: &str, data: &[u8]) -> Self { ... }
    fn path_str(&self) -> &str { ... }
}
impl Drop for TempFile {
    fn drop(&mut self) { let _ = std::fs::remove_file(&self.0); }
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bguthro/seestar-tool](https://github.com/bguthro/seestar-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
