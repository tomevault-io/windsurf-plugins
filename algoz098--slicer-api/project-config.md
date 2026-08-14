---
trigger: always_on
description: Navigation guide for LLMs (coding agents) working in this repository.
---

# AGENTS.md

Navigation guide for LLMs (coding agents) working in this repository.
Read this file BEFORE editing anything.

## GOLDEN RULE — NEVER modify the OrcaSlicer submodule

> **The `OrcaSlicer/` directory is a git submodule of the OrcaSlicer upstream.
> You must NEVER modify, create, or delete files inside `OrcaSlicer/`.**
>
> All of our own logic lives in `OrcaSlicerAddon/` (C++) and `node-api/` (TS).
> If you need behavior different from libslic3r, adapt the layers above
> (`AddonCore`, `EngineAPI`, services), never the submodule.
>
> Before making any change, confirm with `git status` that `OrcaSlicer/`
> does not show up as modified. If it does, revert with
> `git submodule update`.

## What this repository is

A **headless OrcaSlicer wrapper** that exposes the slicing engine (libslic3r)
in two ways:

1. **`orcaslicer-addon`** — native Node.js addon (N-API) published to npm.
   Lets you slice 3D models programmatically from JavaScript.
2. **`node-api`** — HTTP service (Feathers.js + Koa) that wraps the addon and
   exposes REST endpoints for STL/3MF slicing, model introspection, and
   profile conversion.

OrcaSlicer itself is a **git submodule** in `OrcaSlicer/` and provides
`libslic3r` (the C++ library that does the heavy slicing work).

```
HTTP Request  →  Feathers/Koa (node-api, TS)  →  orcaslicer-addon (N-API)
                                                        │  dlopen
                                                        ▼
                                            liborcacli_engine (C FFI)
                                                        │
                                                        ▼
                                              AddonCore (C++) → libslic3r
```

## Repository map

| Path | Description | Language |
|------|-------------|----------|
| `OrcaSlicer/` | Submodule — OrcaSlicer upstream source (libslic3r) | C++ |
| `OrcaSlicerAddon/` | Native addon implementation | C++ + JS |
| `OrcaSlicerAddon/src/core/` | `AddonCore` — high-level facade over libslic3r | C++ |
| `OrcaSlicerAddon/src/engine/` | `EngineAPI` — C FFI layer (`extern "C"`) wrapping `AddonCore` | C++ |
| `OrcaSlicerAddon/src/utils/` | `Logger`, `ErrorHandler` | C++ |
| `OrcaSlicerAddon/bindings/node/` | N-API binding (`addon.cc`) + JS loader + types + Klipper | C++/JS/TS |
| `OrcaSlicerAddon/bindings/node/src/addon.cc` | **N-API entry point** — registers `slice`, `initialize`, etc. | C++ |
| `OrcaSlicerAddon/bindings/node/index.js` | JS loader that resolves the `.node` (prebuilt vs local) | JS |
| `OrcaSlicerAddon/bindings/node/types/index.d.ts` | TypeScript types for the addon API | TS |
| `OrcaSlicerAddon/bindings/node/lib/` | `KlipperClient`, `SliceAndSend` (Moonraker integration) | JS |
| `OrcaSlicerAddon/CMakeLists.txt` | Addon root build (finds OrcaSlicer) | CMake |
| `OrcaSlicerAddon/src/CMakeLists.txt` | Defines `orcacli_core` (static) and `orcacli_engine` (shared) | CMake |
| `OrcaSlicerAddon/bindings/node/CMakeLists.txt` | Defines `orcaslicer_node` (.node module) | CMake |
| `node-api/` | HTTP service (Feathers.js) | TS |
| `node-api/src/app.ts` | Feathers/Koa app setup | TS |
| `node-api/src/orca.ts` | Initializes the addon and injects it into `app.set('orca')` | TS |
| `node-api/src/services/slicer/stl/` | `POST /slicer/stl` — STL slicing → G-code | TS |
| `node-api/src/services/slicer/3mf/` | `POST /slicer/3mf` — 3MF slicing → .gcode.3mf | TS |
| `node-api/src/services/slicer/model-info/` | `POST /slicer/model-info` — model info | TS |
| `node-api/src/services/profiles/` | `GET /profiles` — lists/resolves profiles from resources | TS |
| `node-api/src/services/profile-converter/` | `POST /profile-converter` — converts JSON/ZIP profiles | TS |
| `node-api/src/services/medias/` | Static file service (G-code download) | TS |
| `OrcaSlicerAddon/docs/DEVELOPMENT.md` | Dev doc (LEGACY — describes a CLI that was removed) | — |
| `Makefile` | Orchestrates multi-stage Docker builds | Make |
| `Dockerfile` | Multi-stage build: deps → core → addon → API | Docker |
| `scripts/` | CI and build scripts | Bash |

## Critical design principles

1. **"On-the-fly" mode**: the addon does NOT load vendor bundles at startup.
   All configuration is passed via `options` on each `slice()` call.
   The fallback is `FullPrintConfig::defaults()`. Do not add profile autoloading.

2. **Configuration priority** (highest to lowest):
   `options` (explicit overrides) > `embedded 3MF` > `config`/`profile` > `defaults`.
   - `profile`: base profile applied BEFORE the 3MF is loaded.
   - `options`/`config`: overrides applied AFTER the 3MF (maximum priority).

3. **Engine loaded via `dlopen` at runtime**: the `.node` is thin and loads
   `liborcacli_engine.{dylib,so}` dynamically. This defers the static
   initializers of libslic3r. See `ensure_engine_loaded()` in `addon.cc`.

4. **Serialized operations**: a global `std::mutex` (`g_mutex` in `addon.cc`)
   serializes all heavy calls. Slicing runs in `napi_async_work` but is
   effectively single-threaded at the engine level.

5. **Endpoint security**: the 3MF service ignores the user's `data.output`
   (it generates a name in `os.tmpdir()`) to prevent *arbitrary file write*. The

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [algoz098/slicer-api](https://github.com/algoz098/slicer-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
