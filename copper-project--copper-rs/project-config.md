---
trigger: always_on
description: `copper-rs` is a Rust robotics runtime + SDK built around:
---

# AGENTS.md

## Project Identity

`copper-rs` is a Rust robotics runtime + SDK built around:

- Declarative task graphs in `copperconfig.ron`
- Compile-time runtime generation via `#[copper_runtime(...)]`
- Deterministic execution and replay
- Unified binary logging (`.copper`) plus separate string indexes/logreaders
- A mixed `std` + `no_std` workspace that spans desktop, SBC, and bare-metal targets

The shortest accurate mental model is: "game engine for robots, but the scene graph is a task graph and the engine is generated from RON config."

## Canonical Sources

Read these first when catching up:

1. Root overview: `README.md`
2. CI-aligned commands and local workflows: `justfile`, `CONTRIBUTING.md`
3. Narrative design docs in the sibling wiki checkout: `../copper-rs.wiki/`
4. Tutorial/book material in the sibling book checkout: `../copper-rs-book/book/src/`

Authority order when sources disagree:

1. Code
2. API docs / rustdoc surface
3. Wiki / book narrative docs

Most useful wiki pages:

- `../copper-rs.wiki/Copper-Runtime-Overview.md`
- `../copper-rs.wiki/Copper-Application-Overview.md`
- `../copper-rs.wiki/Copper-RON-Configuration-Reference.md`
- `../copper-rs.wiki/Task-Lifecycle.md`
- `../copper-rs.wiki/Resources.md`
- `../copper-rs.wiki/Project-Templates.md`
- `../copper-rs.wiki/Supported-Platforms.md`
- `../copper-rs.wiki/Available-Components.md`

Most useful book chapters:

- `../copper-rs-book/book/src/ch04-project-structure.md`
- `../copper-rs-book/book/src/ch13-logging-replay.md`
- `../copper-rs-book/book/src/ch18-justfile.md`
- `../copper-rs-book/book/src/ch21-resources.md`

## Workspace Map

- `core/`
  - Main runtime and proc-macro crates.
  - Start with `core/cu29`, `core/cu29_runtime`, `core/cu29_derive`, `core/cu29_traits`.
- `components/`
  - Reusable building blocks grouped as `sources`, `sinks`, `tasks`, `bridges`, `payloads`, `monitors`, `res`, `libs`.
- `examples/`
  - This repo’s best executable documentation. Many features are easier to learn from examples than from API docs.
- `templates/`
  - `cu_project` for single-crate apps, `cu_full` for multi-crate workspaces.
- `support/`
  - CI helpers, cross/deploy support, Docker, `cargo_cubuild`, docs generation helpers.
  - Make examples log into their OWN logs/ subdirectory, NEVER in copper-rs/logs

## Architectural Facts That Matter

- The primary user-facing crate is `core/cu29/src/lib.rs`.
- The `cu29` prelude re-exports nearly everything application authors touch.
- Copper’s design center is static structure, not dynamic runtime discovery.
  - A robot is a static thing.
  - Prefer types, compile-time wiring, and proc-macro/codegen enforcement.
  - Avoid magical runtime string matching, dynamic service discovery, or "things appear later" patterns.
- Runtime generation is proc-macro-driven:
  - `#[copper_runtime(config = "...")]` reads a RON config at compile time.
  - It generates application/builder types and computes the execution plan.
  - Missions in config generate mission-specific builders/modules.
- Config parsing and graph representation live in `core/cu29_runtime/src/config.rs`.
- The proc macro implementation is in `core/cu29_derive/src/lib.rs`.
- `gen_cumsgs!("...")` generates the logreader decode type for a config.
- Logging is a first-class runtime concern, not an optional afterthought:
  - Runtime logs `.copper` binary data
  - Text log strings are interned and reconstructed with `target/debug/cu29_log_index`
  - Replay/resim flows are part of the design, not bolt-ons
- `std` vs `no_std` is a real design axis across the workspace. Do not assume host-only APIs are available everywhere.

## Files To Inspect First For Common Tasks

If the task is about runtime generation:

- `core/cu29_derive/src/lib.rs`
- `core/cu29_runtime/src/config.rs`
- `core/cu29_runtime/src/curuntime.rs`

If the task is about task APIs, message flow, or lifecycle:

- `core/cu29_runtime/src/cutask.rs`
- `core/cu29_runtime/src/cubridge.rs`
- `core/cu29_runtime/src/app.rs`
- `../copper-rs.wiki/Task-Lifecycle.md`

If the task is about resources / HAL wiring:

- `core/cu29_runtime/src/resource.rs`
- `examples/cu_resources_test/`
- `../copper-rs.wiki/Resources.md`

If the task is about logging / export / replay:

- `core/cu29_export/`
- `core/cu29_unifiedlog/`
- `examples/cu_caterpillar/src/logreader.rs`
- `examples/cu_caterpillar/src/resim.rs`

If the task is about templates / DX:

- `templates/cu_project/`
- `templates/cu_full/`
- `templates/README.md`
- Template checks in root `justfile`

If the task is about embedded / no_std:

- `.github/workflows/reusable-embedded.yml`
- `support/ci/embedded_crates.py`
- `examples/cu_rp2350_skeleton/`
- `components/res/cu_micoairh743/`

## Reference Examples

Use these as the quickest way to regain context:

- `examples/cu_caterpillar/`
  - Minimal but realistic Copper app.
  - Good reference for `#[copper_runtime]`, logger setup, determinism, resim, and justfile helpers.
- [`examples/cu_flight_controller/`](https://github.com/copper-project/extra-examples/tree/master/examples/cu_flight_controller)
  - Canonical higher-complexity integration example.
  - Good reference for resources, simulation mode, and real application structure.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [copper-project/copper-rs](https://github.com/copper-project/copper-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
