---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Riker is a fast Rust CLI toolkit that ports key QC metrics tools from Picard/fgbio. It processes SAM, BAM, and CRAM files and produces sequencing QC metrics in TSV format — with lowercase headers, no metadata lines, and `frac_` instead of `pct_` naming conventions.

## Commands

```bash
# Build
cargo build
cargo build --release

# Test (uses nextest)
cargo ci-test                          # all tests
cargo nextest run <test_name>          # single test by name
cargo nextest run --test test_isize    # all tests in one integration test file

# Lint and format
cargo ci-lint                          # clippy with pedantic warnings as errors
cargo ci-fmt                           # check formatting (--check mode)
cargo fmt --package riker              # apply formatting
```

The `ci-*` aliases are defined in `.cargo/config.toml`. CI runs all three.

## Release Packaging

`cargo build --release` and `cargo install riker-ngs` produce **portable**
binaries (no `target-cpu=native`) so users on any reasonable hardware get a
working binary. Distribution channels do the per-platform tuning:

- **x86_64 release / bioconda**: `cargo multivers --profile dist` produces
  a single launcher embedding three CPU variants (`x86-64`, `x86-64-v2`,
  `x86-64-v4`) and dispatches to the highest match at startup. Configured
  via `[package.metadata.multivers.x86_64]` in `Cargo.toml`. The launcher
  is tiny and the embedded variants are delta-compressed.
- **aarch64 release / bioconda**: `cargo build --profile dist`. A single
  portable ARMv8-A binary; `target-cpu=native` showed no measurable win on
  Graviton 4 vs generic, so multivers isn't worth the variant infra.
  Note: do not run `cargo multivers` on aarch64 -- the v1/v2/v3/v4 default
  list is x86_64-only; on aarch64 it would attempt one build per CPU rustc
  knows about (76+), which is slow and pointless given the ~0% gain.
- **`[profile.dist]`** inherits `release` with `incremental = false` for
  deterministic multivers delta-compression. Profiles can't carry
  rustflags on stable Rust, so per-variant `-C target-cpu=...` is supplied
  by `cargo multivers` itself.

For local benchmarking on the host's full ISA (no distribution concerns):

```bash
RUSTFLAGS="-C target-cpu=native" cargo build --release
```

This is opt-in to keep `cargo install` sensible for users on older silicon.

## Architecture

### Crate Structure

- `riker` — binary + library (`riker_lib`)
- `riker_derive` — proc-macro crate for `#[derive(MetricDocs)]` and `#[multi_options]`

The library is named `riker_lib` (see `[lib]` in `Cargo.toml`). External code and tests reference it as `riker_lib::...`.

### Command Pattern

Each subcommand is a struct implementing the `Command` trait (`src/commands/command.rs`). The `Subcommand` enum in `main.rs` dispatches to them via a `match` in `impl Command for Subcommand`.

To add a new command:
1. Create `src/commands/<name>.rs` with the command struct + options struct + collector + metric structs
2. Add `pub mod <name>;` to `src/commands/mod.rs`
3. Add a variant to `Subcommand` in `src/main.rs` and a match arm in `Command::execute()`
4. Integrate with the `multi` command (add `CollectorKind` variant, flatten `MultiMyOptions`, add `build_collectors` arm)
5. Register metric structs in `src/commands/docs.rs`

See the "Adding a New Command" section in `CONTRIBUTING.md` for the full walkthrough.

### Command Module Ordering

Command modules must follow this ordering convention:

1. **`use` statements**
2. **Constants and type aliases**
3. **Structs/enums, each immediately followed by all its impl blocks:**
   - Options struct (e.g. `ErrorOptions`) + `impl` + `impl Default`
   - Command struct (e.g. `Error`) + `impl` + `impl Command`
   - Collector struct (e.g. `ErrorCollector`) + `impl` + `impl Collector`
   - Metric structs (output row types)
   - Helper structs/enums — ordered higher-to-lower level (if A uses B, A comes first), then by importance to the overall implementation
4. **Module-level functions** (functions operating on primitives or external types)
5. **`#[cfg(test)] mod tests`**

**Impl block rules:**
- Consolidate all inherent methods into one `impl` block per struct; keep each trait impl as a separate block
- Trait impls go immediately after the struct's own `impl` block
- Within an impl block, order methods callers-before-callees (higher in the call stack first); constructors (`new`) always come first
- Functions that naturally belong to a type we own should be methods, not standalone functions

### Collector Pattern

Each command's core logic lives in a `Collector` struct implementing the `Collector` trait (`src/collector.rs`):

```rust
pub trait Collector: Send {
    fn initialize(&mut self, header: &Header) -> Result<()>;
    fn accept(&mut self, record: &RecordBuf, header: &Header) -> Result<()>;
    fn finish(&mut self) -> Result<()>;
    fn name(&self) -> &'static str;
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fulcrumgenomics/riker](https://github.com/fulcrumgenomics/riker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
