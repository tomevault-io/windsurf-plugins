---
trigger: always_on
description: `manim-widget` provides an interactive Manim viewer without rendering video. The Python side captures scene intent as JSON; the JS side replays it using `manim-web` in the browser. Primary target is **marimo**, compatible with any `anywidget` frontend.
---

# AGENTS.md

## Vision

`manim-widget` provides an interactive Manim viewer without rendering video. The Python side captures scene intent as JSON; the JS side replays it using `manim-web` in the browser. Primary target is **marimo**, compatible with any `anywidget` frontend.

---

## Goals (priority order)

- **G1 – Fast iteration.** No video rendering pipeline. Notebook execution → visible playback immediately.
- **G2 – Browser-native interactivity.** Playback and camera interactions run in JS without Python round-trips.
- **G3 – Section-aware navigation.** Each section has an entry snapshot enabling cheap direct jumps — no replaying from the start.
- **G4 – Deterministic data contract.** `spec.json` is the wire contract. Always modify it first, then update code and tests together.
- **G5 – Clear unsupported behavior.** Surface predictable warnings/errors; never degrade silently.

---

## Vocabulary

- **Scene**: full execution of `construct()`.
- **Section**: named region delimited by `next_section()`.
- **State bank**: section-local deduplicated list of serialized mobject states (`states`), addressed by integer `state_ref`.
- **Snapshot**: section-entry map of `mob_id → state_ref`. Enables O(1) section jumps without replaying prior sections.
- **Command stream** (`construct`): ordered section operations — `register`, `remove`, `rebind` and the more complex `animate`.
- **Dry-run**: execute scene logic to capture playback data only; no video output.

---

## Architecture

### Python (`src/manim_widget/`)

- **`widget.py`** — Defines `ManimWidget` and the `data` trait payload (`SceneData`). Owns section lifecycle, emits section snapshots and command streams. Uses renderer registry for section-entry snapshots.
- **`renderer.py`** — Custom capture renderer integrated with Manim's `Scene.play` lifecycle. Emits commands and animation descriptors. Maintains per-section deduplicated state banks and allocates `state_ref` values. Handles `rebind` for replacement-style transforms.
- **`snapshot.py`** — Short-id generation and mobject serialization primitives.

### JavaScript (`js/src/`) — **edit source here, never `src/manim_widget/static/index.js` directly**

- **`index.js`** — anywidget entry point. Creates scene, registry, player, wires controls.
- **`registry.js`** — Runtime mobject registry keyed by stable IDs.
- **`player.js`** — Restores section snapshots and executes command streams. Resolves `state_ref` through `section.states` before restoring mobjects. Animation adapter must handle both constructor-style and factory-style exports from `manim-web`.
- **`test_cli.js`** — CLI integration test entry point. Uses `happy-dom` for a browser-like environment and `manim-web` headless mode (scene graph only, no pixel output). Reads scene spec from stdin; always outputs `sectionIds` (registry + scene IDs per section) and `sectionEndStates` in the result JSON.

### Bundled runtime (`src/manim_widget/static/index.js`)

Built from `js/src/*` via Bun with the typia transform applied. This is what packaged widget users execute. Rebuild after any JS source change with `cd js && bun run build.ts`.

### manim-web

`manim-web` is a git submodule at `manim-web/` (repo root). **Always read its source from `manim-web/src/`, never from `js/node_modules/manim-web/`.** The node_modules copy is a build artifact and may be stale. Upstream PRs are sent when bugs are found. The JS side of `manim-widget` may eventually move into `manim-web` directly, making the JSON spec a first-class concept there.

---

## Data Contract

`spec.json` is the single source of truth. When changing payload shape: update `spec.json` first, then code, then tests.

The spec is deliberately minimal — favor the smallest set of primitives that can express a behavior over adding a dedicated shape for it. Before adding a new field/kind, check whether an existing primitive already composes into it.

### Top-level shape

```json
{
  "version": 2,
  "states": [ { "kind": "VMobject", ... }, ... ],
  "sections": [ ... ]
}
```

- `states`: global deduplicated bank shared across all sections. All commands, frames, and snapshots reference entries by integer index.
- Think of the state bank as a DAG, not a flat list: `state_ref`s are dependency edges (e.g. `GroupState.children`, `Derived.from`, `MathTexSource` transform corners). This is what gives dedup its power — a state is defined in terms of the states it depends on, much like bindings in a functional language.

### Section

```json
{
  "name": "intro",
  "snapshot": { "mob_id": 0 },
  "construct": [ ... ]
}
```

- `snapshot`: `mob_id → state_ref` into the global `states` bank, for all root mobjects at section entry. Group children are **not** listed separately — they are referenced via `GroupState.children`.
- `construct`: ordered command list.

### Commands

| cmd | purpose |
|---|---|
| `register` | bind `id → state_ref` in scene graph and show in scene; idempotent — if `id` is already registered, updates its state (incl. `fixed` status) in place instead of recreating it |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rambip/manim-widget](https://github.com/rambip/manim-widget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
