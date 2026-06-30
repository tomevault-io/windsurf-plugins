---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a pure Rust implementation of OpenUSD (Universal Scene Description), Pixar's open-source framework for 3D computer graphics data interchange. The project aims to provide native Rust support without C++ dependencies for reading and writing USD files.

## Architecture

The codebase mirrors the C++ OpenUSD SDK's module layout. The bullets below are only a navigational map — what each module is and where to start reading. They deliberately do not enumerate types or methods: the project is large, and any inventory kept here would drift out of date. Each module's own `//!` doc comment, and each item's own doc comment, are the source of truth — read those for specifics rather than expecting them here.

- **`tf/`** - Tools Foundation (C++ `Tf`): low-level utilities, chiefly `tf::Token`, the interned-identifier string behind every `TfToken`-equivalent API. Start at `tf/mod.rs`.

- **`sdf/`** - Scene Description Foundations (C++ `Sdf`): the core data model. `AbstractData` is the unified read/write backend interface the text, binary, and archive readers all implement; `Value`, `Path`, and `Layer` (C++ `SdfLayer`) are the everyday types. A `Layer`'s edits run through `Layer::edit` (a closure over a `LayerEdit` view) as atomic copy-on-write transactions that derive a `ChangeList` for composition invalidation and fire per-layer `LayerSink`s at the commit seam (`sdf/layer.rs`, `sdf/change.rs`, `sdf/sink.rs`); spec authoring lives on the typed views (`PrimSpec::new` and friends). Also here: the pluggable `FileFormat` read/write seam, the `LayerRegistry` that owns asset loading and format dispatch (composition opens reference/payload targets on demand through it, so an un-visited subtree never loads), the variable-expression engine (`sdf/expr.rs`), and the namespace-aware spec-copy primitive (`sdf/copy.rs`). Start at `sdf/mod.rs`.

- **`usda/`** - Text format `.usda`: logos lexer + recursive-descent parser. `TextReader` / `TextWriter`.

- **`usdc/`** - Binary crate format `.usdc` (compressed). `CrateData` / `CrateWriter`.

- **`usdz/`** - ZIP-based `.usdz` package. `Archive` / `ArchiveWriter`.

- **`ar/`** - Asset Resolution (C++ `Ar`): the `Resolver` trait maps `@...@` asset paths to physical locations; `DefaultResolver` searches the filesystem.

- **`pcp/`** - Prim Cache Population, the composition engine (C++ `Pcp`): LIVERPS strength ordering across layers, kept a pure function of `(graph, context, cached indices)` so it stays parallelizable. Composition drives layer loading: a reference/payload target opens on first use and the demand travels out through `BuildOutput` to the stage's load barrier. Start at `pcp/mod.rs` — it has the LIVERPS overview and a per-file structure table.

- **`usd/`** - Composed stage API (C++ `Usd`): `usd::Stage` is the handle that delegates composition to `pcp::IndexCache`; `Prim`, `Attribute`, `Relationship`, and the schema views are `Clone` value types over it, and stage-tier authoring routes through the current `EditTarget`, with `Stage::batch_edit` for atomic multi-layer edits. Notable sub-surfaces: `usd/sink.rs` (`StageSink` composed-change observers, `Provenance`), `usd/diff.rs` (edit replication), `usd/editor.rs` (namespace editing). Start at `usd/stage.rs`. Public users import modules (`use openusd::{sdf, usd};`), not root-level re-exports.

- **`schemas/`** - Domain schemas layered on `sdf` / `usd`, not part of the AOUSD core spec. Feature-gated per family (`geom`, `lux`, `media`, `physics`, `proc`, `render`, `shade`, `skel`, `ui`, `vol`; some enable `geom` transitively). See the table in `schemas/mod.rs`.

- **`gf/`** - Graphics Foundations (C++ `Gf`): `bytemuck::Pod` vector / quaternion / matrix types for bulk binary serialization, row-major / row-vector convention, each bridging to `sdf::Value` via `From` / `TryFrom`. See `gf/mod.rs` for the conventions.

## Development Commands

```bash
# Build the project (use --all-features to include the gated schema modules)
cargo build --all-features

# Run tests (including comprehensive format validation tests)
cargo test --all-targets --all-features

# Lint with Clippy (strict warnings as errors)
cargo clippy --all-targets --all-features -- -D warnings

# Format code
cargo fmt

# Check formatting
cargo fmt --all -- --check --files-with-diff

# Generate documentation
cargo doc --no-deps

# Run security/dependency audits
cargo deny check

# Run examples
cargo run --example dump_usdc -- path/to/file.usd
```

## ROADMAP.md Style

Keep `ROADMAP.md` rows concise. Each Notes cell should cover three things only:

1. Whether it is done — the status emoji in the Status column handles this;
   the Notes cell does not need to restate it.
2. What to read — name the key types or traits a reader would look for
   (e.g. `sdf::ChangeList`, `usd::NamespaceEditor`). Do not describe what
   those types do; their own doc comments are the source of truth.
3. What is remaining — a short `Remaining — X; Y` clause, or a `Remaining:`
   bullet list when there are several distinct items, for anything materially
   incomplete. Drop it once nothing is left.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mxpv/openusd](https://github.com/mxpv/openusd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
