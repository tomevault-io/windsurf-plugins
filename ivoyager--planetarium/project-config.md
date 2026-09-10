---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

I, Voyager Planetarium — an open-source 3D solar system simulator built on **Godot Engine 4.7+** using **GDScript**. Displays accurate orbital mechanics for planets, moons, spacecraft, and ~70k asteroids. Runs as a Windows desktop app or Progressive Web App.

## Running the Project

Open in Godot Editor and press Play. No external build system — Godot handles everything.

**First-time setup:** Clone with `--recursive` for submodules. The editor plugin auto-downloads assets (~216 MiB) on first run — press "Download" when prompted.

**Export targets** (defined in `export_presets.cfg`):
- Web: `export/planetarium-rc.html` (PWA, single-threaded). `variant/thread_support` is off and
  stays off: SharedArrayBuffer needs cross-origin isolation headers, which is more than we will
  ask of a host. Don't propose threading as a fix for anything.
- Windows: `export/Planetarium-v0.1.exe` (x86_64)

There is no test framework or linter beyond Godot's built-in GDScript warnings.

**Shader compile time is the dominant first-run cost on the Compatibility renderer, and so in
the web export** — a cold start spends anywhere from tens of seconds to a couple of minutes
compiling, depending on the GPU, nearly all of it now behind the boot screen via the Core
plugin's `IVShaderWarmup`. `addons/ivoyager_core/SHADER_COMPILE_COST.md` carries the per-shader
measurements, what actually drives them (the GL compiler unrolling constant-bound loops, not
source length or include weight), why the Compatibility light configuration is the largest
remaining lever, what editing a given `.gdshaderinc` costs in recompiles, where the two shader
caches live, and how to measure it again. Read it before touching a shader or hand-unrolling
anything. The timing harness is `addons/tools/time_shader_compiles.py`, run from this directory.

### GDScript Warning Preferences

All GDScript code should compile with **zero warnings**. Apply these strategies:

- **UNSAFE_CALL_ARGUMENT / UNSAFE_METHOD_ACCESS / UNSAFE_PROPERTY_ACCESS** — Fix by editing code. For built-in types, assign the Variant to a properly typed intermediate variable before passing it to a typed function parameter or constructor (e.g., `int()` requires `int`/`float`/`bool`, not `Variant`). Note: `as ClassName` generates UNSAFE_CAST — avoid it; direct assignment from `Object`-typed dictionary `.get()` to a typed member variable does not warn.
- **UNUSED_VARIABLE** — Prefix with `_` (e.g., `for _k in count:`).
- **INTEGER_DIVISION** — Suppress with `@warning_ignore("integer_division")` where integer division is intentional.
- **SHADOWED_VARIABLE** — Suppress with `@warning_ignore("shadowed_variable")` only in static functions where shadowing the instance variable is expected. In all other cases, rename the variable to avoid shadowing.

## Architecture

### Design documents

Three documents in `addons/ivoyager_core/` describe the simulation at the level of logic and
invariants, each with its own TODO list. Read the relevant one before changing that subsystem:

- `PHYSICAL_MODEL.md` — the objective simulation: bodies, orbits as an element coordinate
  system, trajectories, rotation, time, units and scale, small-body groups, the persisted
  state and what a multiplayer sync would need.
- `VISUAL_MODEL.md` — how that double-precision truth renders through a float32 pipeline:
  parenting, origin shifting, farwarp, the shadow systems, culling, orbit lines, point fields,
  mouse picking.
- `PHOTOMETRIC_MODEL.md` — physically calibrated light: the calibration chain, the
  compensating camera, surfaces, atmospheres, rings, stars, renderer parity.

`addons/ivoyager_core/IVBody_REDESIGN_v0.3.md` is the living plan for the IVBody rework;
`PHYSICAL_MODEL.md` and `VISUAL_MODEL.md` link to it. When the redesign lands and that file
goes away, remove those links.

### Plugin System (Git Submodules)

The core simulation lives in three plugins under `addons/`, each a git submodule:

- **ivoyager_core** — Orbital simulation engine, 3D rendering, camera, UI widgets, singletons
- **ivoyager_tables** — CSV-based data table import system (planet/moon/asteroid data)
- **ivoyager_units** — Unit conversion system (template replaced by `planetarium/units.gd`)

Two more submodules support development:

- **ivoyager_assistant** — In-sim TCP/JSON-RPC server for AI-driven testing (see *Testing with the Assistant Plugin*)
- **tools** — Python asset-generation and data-conversion scripts; not a Godot plugin (see *Asset & Data Pipelines*)

A further directory, `addons/ivoyager_assets/`, holds 3D models and textures (not Git-tracked, downloaded by the editor plugin). It is also the deploy target of a separate private build tree — see *Changing a distributed asset* below before writing anything into it.

### Planetarium Shell (`planetarium/`)

This repo is a thin "shell" that configures and extends the core plugins:

- `universe.gd` / `universe.tscn` — Main scene root (extends `Node3D`)
- `preinitializer.gd` — Primary configuration entry point: sets `IVCoreSettings`, registers program objects, configures timekeeper and speed manager

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivoyager/planetarium](https://github.com/ivoyager/planetarium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
