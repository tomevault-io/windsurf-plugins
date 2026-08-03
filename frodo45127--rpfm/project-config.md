---
trigger: always_on
description: This document provides guidelines for contributing to the Rusted PackFile Manager (RPFM) codebase. These rules MUST be followed by all AI coding agents and contributors.
---

# Agent Guidelines for RPFM

This document provides guidelines for contributing to the Rusted PackFile Manager (RPFM) codebase. These rules MUST be followed by all AI coding agents and contributors.

## Project Context

RPFM is a Rust + Qt6 desktop application for opening, inspecting, editing and saving PackFiles for every Total War since *Empire: Total War*.

Workspace layout:

### Libraries

- `rpfm_lib` — Core file-format library: packs, schemas, DB, Loc, RigidModel, audio, video, etc.
- `rpfm_extensions` — Higher-level workflows: dependencies, diagnostics, search, optimizer, translator, glTF export.
- `rpfm_ipc` — Command/response protocol shared between UI and server.
- `rpfm_telemetry` — Logging, crash reporting and opt-out action telemetry (Sentry-backed).
- `rpfm_ui_common` — Shared Qt6 helpers used by every UI consumer.

### Executables

- `rpfm_ui` — The Qt6 desktop application most users run. Depends on Qt6 bindings under `3rdparty/`.
- `rpfm_server` — Headless backend doing the heavy file/schema/filesystem work. Exposes WebSocket + MCP endpoints. The UI spawns it automatically.

The `3rdparty/` directory is excluded from the workspace. It contains vendored `ritual`-generated Qt6 bindings (`qt_core`, `qt_gui`, `qt_widgets`, `qt_ui_tools`, `cpp_core`). **Do not touch files in `3rdparty/` (except `3rdparty/src/qt_rpfm_extensions`) unless explicitly asked.**

MSRV is `1.85`, edition `2021`.

## Your Core Principles

Code you write MUST be clear, correct, and appropriately optimized for the task:

- Maximize algorithmic big-O efficiency for memory and runtime where it matters (PackFile decoding/encoding, diagnostics, search).
- Use `rayon` for CPU-bound parallelism when iterating over large collections (tables, files within a pack).
- Follow idiomatic Rust and maximize code reuse (DRY).
- No extra code beyond what is necessary to solve the problem. No speculative abstractions.

If you believe code is not ready to hand off, do another pass before stopping.

## Preferred Tools and Crates

The workspace already pins the canonical set of dependencies in the top-level `Cargo.toml`. When adding functionality, prefer what's already there before introducing new crates:

- Building / dependencies: `cargo` via the workspace `Cargo.toml`.
- Error handling: `thiserror` for library error types, `anyhow` for application-level errors / glue code.
- Serialization: `serde` with `serde_json`, `ron` (schemas and settings), `bitcode` (compact binary encoding), `toml` where applicable.
- Logging: the `log` crate (funneled through `rpfm_telemetry`). Use `log::error!`, `log::warn!`, `log::info!`, `log::debug!`, `log::trace!` instead of `println!` / `eprintln!` / `dbg!`.
- Async runtime: `tokio` (used mainly by `rpfm_server` and `rpfm_ipc`).
- Parallelism: `rayon` for CPU-bound work; `crossbeam` channels for threaded message passing.
- Accessors: `getset` for generated getters/setters on structs with many private fields (matches existing style).
- HTTP: `reqwest` (the `blocking` feature is used in parts of the codebase; match the surrounding module's style).
- Self-update: `self_update` (already wired up, do not reinvent).
- Qt6 GUI: `qt_core`, `qt_gui`, `qt_widgets`, `qt_ui_tools` via the vendored `ritual` bindings. Qt interop goes through `cpp_core` (`CppBox`, `Ptr`, `MutPtr`).
- MCP (server only): `rmcp` with `schemars` for tool schemas.

## Code Style and Formatting

- **NEVER** run `rustfmt` / `cargo fmt`. The maintainer has explicitly rejected it — past experiments grew the codebase from ~18k to >30k lines of noise. Match the surrounding code's manual formatting instead.
- Clippy-suggested fixes are welcome. Run `cargo clippy` and apply reasonable suggestions.
- **MUST** use 4 spaces for indentation (never tabs).
- **MUST** use meaningful, descriptive variable and function names.
- Use `snake_case` for functions/variables/modules, `PascalCase` for types/traits, `SCREAMING_SNAKE_CASE` for constants.
- Follow Rust API Guidelines and idiomatic Rust conventions.
- **NEVER** use emoji, or unicode that emulates emoji (e.g. check/cross glyphs). The only exception is when tests need to exercise multibyte characters.

## Comments and "No Black Magic"

> **HARD RULE — NO COMMENTS LONGER THAN 2 LINES INSIDE FUNCTIONS.** No exceptions. If a longer explanation is genuinely needed, lift it into the function's doc comment instead (doc comments may be longer). This is the most-violated rule here — re-read it before writing any in-body comment.

Per `CONTRIBUTING.md`: **explain what your code does** and **no black magic code**. The maintainer uses this project to learn Rust, so unexplained cleverness is a regression.

- When using a non-obvious Rust feature (lifetime tricks, complex trait bounds, `unsafe`, manual `Drop`, custom `Deref`), add a short comment explaining *why*.
- Comments should describe intent and rationale, not restate what the code obviously does.
- Minimize comments inside function bodies and don't overexplain: add one only where the code isn't self-evident.
- Keep comments up to date with code changes. Delete stale ones.

## Documentation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Frodo45127/rpfm](https://github.com/Frodo45127/rpfm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
