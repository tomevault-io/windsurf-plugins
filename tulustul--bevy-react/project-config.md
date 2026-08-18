---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`bevy-react` drives **`bevy_ui` from a React app** running on an embedded V8 (deno_core) runtime. It is a **Rust crate + JS package** (one repo) plus an example app. React renders through a custom reconciler that emits UI-mutation ops; Rust applies them to the Bevy ECS; interactions and app-level messages flow back.

This is a virtual Cargo workspace with two crates under `crates/`: `core` (the `bevy-react` lib) and `macros` (the `bevy-react-macros` proc-macro crate). The canvas rasterizer, SVG subsystem, animation engine, and portal/surface host elements are public modules of `core` (`crates/core/src/{canvas,svg,animations,portal.rs,surface.rs}`). It is **also** an npm workspace (root `package.json` with members `js` and `examples/demos/ui`). The two halves are developed together. The example (`examples/`) and the JS runtime (`js/`) stay at the repo root; `core` declares the `demos` example via an explicit `[[example]] path`.

The example (`examples/demos`) is a gallery: a left-nav switches between demos, and React drives the active 3D scene with `bevy.selectScene(id)` (or `null` for an empty viewport). The three scenes each live under `examples/demos/scenes/` as a **separate Bevy plugin**: `cubes.rs` (cubes driven by `emit`), `bouncing_ball.rs` (one ball that both pushes Bevy→React events as toasts and answers request/response polls, and owns its own ball physics), and `crowded_cubes.rs` (crowded cubes with world-anchored badges). Only one scene runs at a time: React `emit`s the selection, and a `States` enum (`Scene` in `examples/demos/scene.rs`, variants `None | Cubes | BouncingBall | CrowdedCubes`) gates each scene's systems, with `DespawnOnExit(Scene::…)` scoping its entities. The shared 3D camera (auto-orbit + drag + zoom) is bundled as `CameraPlugin` in `examples/demos/camera.rs`.

## Commands

Build / run the example (this is the main way to see it working):

```sh
npm install                          # once
npm run build -w demos           # build the React bundles — REQUIRED before running
cargo run -p bevy-react --example demos   # run the Bevy app (needs a GPU/window)
npm run watch -w demos           # rebuild app bundle on change → React Fast Refresh
```

The build (`examples/demos/ui/build.mjs`, via `bevy-react/build-lib`) emits **two**
bundles into `dist/`: `vendor.js` (react + react-reconciler + the bevy-react runtime,
loaded into the isolate once and never re-run) and `app.js` (the app's own components,
an IIFE re-executed on each edit). Editing a component preserves its `useState`/hook
state — see "Hot reload: React Fast Refresh" below.

Tests:

```sh
cargo test                           # all Rust tests (whole workspace)
cargo test -p bevy-react --lib       # just the core library unit tests
cargo test -p bevy-react --lib ts_codegen::tests::exports_typescript   # a single test by path
cargo test -p bevy-react --test roundtrip   # headless end-to-end bridge test (real JS runtime)
```

The `roundtrip` test drives the JS thread directly and asserts an initial render + click round trip. **It requires the bundle to be built first** (`npm run build -w demos`); if the bundle is missing it skips (passes) with a notice.

Lint / format / typecheck (run from repo root):

```sh
npm run lint          # eslint + tsc (all workspaces) + format:check — NOT clippy
cargo clippy --workspace --all-targets    # run separately (CI runs it as its own step)
npm run typecheck     # tsc --noEmit across JS workspaces
npm run format        # prettier --write . && cargo fmt
npm run format:check  # CI-style check (prettier + cargo fmt --check)
```

`npm run lint` is JS-side only (plus `cargo fmt --check` via `format:check`); clippy is deliberately **not** in it — CI's typescript job runs `npm run lint` on a runner without Bevy's system deps, so clippy lives in the rust job as its own step. Run it yourself before handing off Rust changes.

## Architecture: the Rust↔JS bridge

The whole boundary is a dedicated **JS thread** (owns the V8 isolate, runs a `current_thread` tokio runtime) connected to the **Bevy main thread** by channels. Understanding the message flow requires reading `crates/core/src/js_thread.rs`, `crates/core/src/plugin.rs`, `crates/core/src/protocol.rs` (a directory module — `protocol/{op,props,merge,style,…}.rs`), and `js/src/bridge.ts` together.

**Four ops** (`crates/core/src/js_thread.rs`) form the boundary:

- `op_flush(ops)` — JS→Bevy, sync. The reconciler batches a `Vec<Op>` per commit and flushes it. `serde_v8` deserializes straight into `protocol::Op` — no JSON strings on the hot path.
- `op_emit(name, value)` — JS→Bevy, sync, fire-and-forget app message.
- `op_request(id, name, value)` — JS→Bevy, sync; correlated request awaiting a reply.
- `op_next_event()` — Bevy→JS, **async** (Rust Future → JS Promise). Returns one `protocol::Outbound`.

**Channels** are created in `ReactUiPlugin::build` (`crates/core/src/plugin.rs`): crossbeam for JS→Bevy (`Vec<Op>`, `ReactMessage`, `RawRequest`), tokio mpsc for Bevy→JS (`Outbound`) and reload signals.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tulustul/bevy-react](https://github.com/tulustul/bevy-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
