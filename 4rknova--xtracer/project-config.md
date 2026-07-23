---
trigger: always_on
description: This file is the orientation and operating guide for coding agents working in this repository.
---

# AGENTS.md

This file is the orientation and operating guide for coding agents working in this repository.

## Project Snapshot

- Name: `xtracer`
- Language: C/C++ (CMake)
- Branch context: this workspace is currently on `develop`.
- Core purpose: experimental rendering framework with a shared rendering core and multiple frontends (CLI, Web, WASM).

## Repository Map

- Core renderer and scene system: `src/xtcore/`
- CLI frontend: `src/frontend/cli/`
- Web frontend backend: `src/frontend/web-server/`
- WASM frontend runtime: `src/frontend/wasm/`
- Frontend shared helpers: `src/frontend/common/`
- Web static assets: `src/frontend/web-client/`
- Supporting libraries: `lib/`
- Third-party dependencies: `ext/`
- Scene examples: `scene/`
- Build definition: `CMakeLists.txt`

## Build Notes

- Historically documented flow in README:
  - `./configure`
  - `make`
- Practical modern flow is CMake-driven.
- Web build is optional (see `XTRACER_ENABLE_WEB` usage in `CMakeLists.txt`).
- WASM build is optional (see `XTRACER_ENABLE_WASM` usage in `CMakeLists.txt`).
- In this workspace, in-source CMake artifacts exist (`CMakeCache.txt`, `CMakeFiles/`, etc.). Prefer out-of-tree builds for new runs.

## Known Current State (Develop)

- Integrator implementations exist under `src/xtcore/integrator/`.
- Integrator metadata exposed by web/WASM is defined in `src/frontend/common/render_service.*`.
- CLI integrator selection code in `src/frontend/cli/xtracer.cc` is currently commented out; treat CLI rendering path as needing repair before relying on it.
- Web frontend is implemented as `xtracer_web`:
  - HTTP server via `ext/cpp-httplib/httplib.h`
  - Async render jobs managed in `src/frontend/web-server/job_manager.*`
  - Shared render pipeline in `src/frontend/common/render_service.*`
  - Backend log stream in `src/frontend/web-server/backend_log.*`
  - Static SPA in `src/frontend/web-client/` with tabs: `Render`, `Editor`, `Settings`, `Logs`, `About`

### Web API Surface (Current)

- `GET /api/health`
- `GET /api/about`
- `GET /api/scenes`
- `GET /api/scenes/{scene}/cameras`
- `GET /api/scenes/{scene}/source`
- `GET /api/scenes/{scene}/geometry`
- `GET /api/scenes/{scene}/runtime_graph`
- `GET /api/scenes/{scene}/camera_resolve`
- `GET /api/scenes/load_jobs/{id}`
- `POST /api/scenes/save`
- `GET /api/integrators`
- `POST /api/render`
- `GET /api/jobs/{id}`
- `GET /api/jobs/{id}/image`
- `GET /api/logs?since=<id>`

### Web Runtime Notes

- `xtracer_web` defaults:
  - host: `127.0.0.1`
  - port: `8080`
  - scene dir: `scene/`
  - web root: `src/frontend/web-client/`
- Job execution is serialized via a global render mutex in web backend (avoids OpenMP oversubscription from concurrent jobs).
- PNG responses are currently produced by rendering to `nimg::Pixmap` then encoding via temporary file path.

## Branch Relationship Reminder

See: `docs/BRANCH_RELATIONSHIP.md`

Short version:

- `master` and `develop` diverged years ago.
- `develop` contains newer 2024 rendering/build cleanup commits (remotery removal and optional GUI build).
- `master` contains several repo housekeeping commits not present on `develop`.

## Task-Specific References

- Architecture notes: `docs/ARCHITECTURE_NOTES.md`
- Renderer/integrator inventory: `docs/RENDERERS.md`
- Web UI/feature behavior: `src/frontend/web-client/index.html`, `src/frontend/web-client/app.js`, `src/frontend/web-client/styles.css`
- Web widget library: `src/frontend/web-client/app/widgets/`
- Widget showcase (live reference): `src/frontend/web-client/showcase.js`

## Scene Format Reference (Parser-Backed)

Source: `src/xtcore/parseutil.cc`, `src/xtcore/proto.h`. See also `scene/` for real examples and `.claude/commands/create-scene.md` for the interactive creation skill.

### Value Types

| Type | Syntax | Example |
|------|--------|---------|
| bool | `true` / `false` / `1` / `0` | `flip_normals = true` |
| int | decimal | `resolution = 64` |
| float | decimal or scientific | `radius = 1.5` |
| string | quoted or bare | `source = "path/file.obj"` |
| col3 | `col3(r,g,b)` | `col3(1,0.5,0)` — values 0–1 typical; higher for HDR/emissive |
| vec3 | `vec3(x,y,z)` | `vec3(0,1,0)` |
| tex2 | `tex2(u,v)` | `tex2(0.5,0.5)` |

Coordinate system: **right-handed, Y-up**. Modifier order when applied to meshes: rotation → scale → translation.

### Top-Level Structure

```
title       = "My Scene"
description = "Optional"
version     = "1.0"

environment = { type = ..., config = { ... } }
camera      = { cam_name = { type = ..., ... } }
geometry    = { geom_name = { type = ..., ... } }
material    = { mat_name  = { type = ..., properties = { samplers = { ... }, scalars = { ... } } } }
object      = { obj_name  = { geometry = geom_name, material = mat_name } }
medium      = { med_name  = { ... } }   # optional, for volumetrics
variants    = { var_name  = { set = { ... }, remove = { ... } } }  # optional
```

### Environment Types

| Type | Required config fields |
|------|----------------------|
| `gradient` | `a` col3, `b` col3 |
| `color` | `value` col3 |
| `erp` | `source` string (path to .hdr) |
| `cubemap` | `posx/posy/posz/negx/negy/negz` strings |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [4rknova/xtracer](https://github.com/4rknova/xtracer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
