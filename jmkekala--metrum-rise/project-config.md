---
trigger: always_on
description: Metrum Rise is a large-scale city simulation game. The long-term goal is to support **≥1,000,000 total population across simulation tiers**. Full-FSM simulation is reserved for the active area of interest; distant parts of the same world may run at coarser aggregate fidelity. World scale is flexible, with 20 km × 20 km as the default working scale.
---

# Metrum Rise — CLAUDE.md

## Project Overview

Metrum Rise is a large-scale city simulation game. The long-term goal is to support **≥1,000,000 total population across simulation tiers**. Full-FSM simulation is reserved for the active area of interest; distant parts of the same world may run at coarser aggregate fidelity. World scale is flexible, with 20 km × 20 km as the default working scale.

**Architecture:** Rust simulation backend compiled as a GDExtension DLL (`libmetrum_rise.so`), loaded by a Godot 4 frontend that handles rendering and user input.

## Tech Stack

Current stack summary. For exact versions, check `rust/Cargo.toml` and the Godot project/runtime.

- Godot 4.x frontend, Rust simulation backend, `godot-rust` / gdext bridge
- Rayon for parallelism
- SQLite (`rusqlite`) plus serde-based data formats
- Criterion for benchmarks
- Blender for modeling

## Project Structure

- `rust/src/simulation/` contains the core simulation systems: network, pathing, buildings, economy, grids, and save/load.
- `rust/src/nodes/` contains the Godot bridge and runtime-facing APIs.
- `rust/src/assets/` contains asset packs, registry, and validation.
- `godot/scripts/` contains thin UI/input/render bridges.
- `docs/` contains the dashboard, roadmap, reference tables, subsystem specs, and archive.
- `$HOME/.local/share/godot/app_userdata/Metrum\ Rise/mods/` contains all the user assets

## Building and Running

```bash
./run.sh              # Debug build, deploys .so, launches Godot
./run.sh --headless   # Headless mode
cd rust && cargo build --release   # Release build
cd rust && cargo bench             # Criterion benchmarks
```

The compiled library must be at `godot/bin/libmetrum_rise.so`. `run.sh` handles this automatically.

## Performance Philosophy

**This project is performance-first.** The 1M-agent scale target is non-negotiable and must be kept in mind for every decision, including small ones. Correctness without acceptable performance is not a done state.

- Determinism is the default. Given the same save state, inputs, and tick sequence, the simulation should produce the same results. Any intentional randomness must be explicit, controlled, and justified; cosmetic variance should not silently change simulation outcomes. Heuristic solutions are not tolerated.
- Measure before you add. Every new system must have a clear complexity bound. If a proposed implementation would degrade an existing O(1) or O(log N) path to O(N) or worse at city scale, it is not acceptable.
- Reuse before you build. Before writing new data structures, algorithms, or abstractions, check whether an existing one already solves the problem. `DataGrid<T>`, the road-edge `rstar` R-tree, the 16 m node lookup grid, the 512 m building/routing chunk indices, the SoA agent layout, and Rayon parallelism cover the majority of simulation needs. Adding another spatial structure when one of those already answers the query is a maintenance cost with no benefit.
- Hot-path allocations are bugs. Any allocation inside a per-tick or per-agent loop is a correctness issue at scale, not a style issue.
- Parallelism is the default, not an optimisation. If a system iterates over a flat collection independently per element, it uses `rayon::par_iter`. Single-threaded iteration over large collections is a conscious exception that must be justified.

## Bugs and Backlog

- `docs/project.md` is the live dashboard: current status, current focus, recent changes, and links to the owning docs.
- `docs/roadmap.md` is the live work tracker: stable IDs, active priorities, validated bugs, and later tracks.
- Update the dashboard when a system changes materially, and update the roadmap when tracked work status changes.
- Do not hide bugs behind workarounds. Fix the root cause and update the dashboard and roadmap as needed.
- Use stable IDs in docs and notes. Do not introduce fresh positional references like `item 30`.

## Documentation Practices

- **`docs/README.md`** — docs index and ownership map. Start there when deciding where a change belongs.
- **`docs/project.md`** — current dashboard for what is shipped, what is being focused on, and where the owning docs live. Keep it summary-first.
- **`docs/roadmap.md`** — active tracked work, stable IDs, and later priorities. Do not use positional backlog numbering in new docs.
- **Subsystem docs in `docs/`** — files such as `entrance_and_exit.md`, `economy.md`, `demand.md`, and `zoning.md` own the detailed design/spec contracts for their respective systems. When behavior changes, update both the subsystem spec and the relevant status note in `project.md`.
- **`CLAUDE.md` / `AGENTS.md`** — contributor guidance, architectural invariants, workflow rules, and sharp edges for coding agents. Keep this file focused on stable guidance rather than exhaustive feature-by-feature implementation history.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmkekala/metrum_rise](https://github.com/jmkekala/metrum_rise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
