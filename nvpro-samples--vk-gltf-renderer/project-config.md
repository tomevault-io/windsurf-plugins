---
trigger: always_on
description: Read this first. It is a **router**, not a manual: it points you at the right
---

# CLAUDE.md — Agent guide for vk_gltf_renderer

Read this first. It is a **router**, not a manual: it points you at the right
doc and states the rules to follow. Keep it thin — deep knowledge lives in
`docs/`. If you change an architectural invariant, update the relevant `docs/`
file **and** this file in the same commit.

## What this is

A Vulkan **ray tracing** renderer for glTF 2.0 scenes, built on
[nvpro_core2](https://github.com/nvpro-samples/nvpro_core2) with
[Slang](https://github.com/shader-slang/slang) shaders. Two render paths share
the same Vulkan resources and scene data:

- **RTX path tracer** (`renderer_pathtracer`) — the authoritative, high-quality
  reference for glTF PBR material evaluation and new extensions.
- **Rasterizer** (`renderer_rasterizer`) — a fast preview path for interaction
  and editing. Not the material reference.

C++20. Host and device share structs via `shaders/shaderio.h`.

## Where to read before you code (do this instead of grepping the whole tree)

| Your task | Read |
|---|---|
| Understand overall architecture, app lifecycle, source map | [docs/developer.md](docs/developer.md) |
| Scene data flow: Model → RenderNodes → GPU SSBO / BLAS / TLAS | [docs/RENDERING_ARCHITECTURE.md](docs/RENDERING_ARCHITECTURE.md) |
| glTF 2.1 multi-file scenes, read-only flagging, re-externalize on save | [docs/external_assets.md](docs/external_assets.md) |
| Runtime behavior, editor workflows, features | [docs/user-guide.md](docs/user-guide.md) |
| DLSS / OptiX denoising, motion vectors, jitter/reset (incl. why animated meshes ghost) | [docs/denoising.md](docs/denoising.md) |
| Headless timing / scripted GPU benchmarks | [docs/benchmarking.md](docs/benchmarking.md) |
| Test suite: running (CTest vs. direct), adding tests, benchmarks | [tests/README.md](tests/README.md) |
| Where scene/model assets come from | [docs/external_assets.md](docs/external_assets.md), [docs/resources.md](docs/resources.md) |

`docs/developer.md` contains the full annotated `src/` and `shaders/` file map —
use it to locate a subsystem instead of fanning out readers across the tree.

## Source of truth — read the code, not the doc

For anything the compiler or loader **enumerates or enforces**, the code is authoritative and
docs drift. Grep the symbol below instead of trusting a prose list, and when a doc disagrees
with the code, the code wins (fix the doc — see "Keep the docs true").

| You need… | Grep / read this (authoritative) |
|---|---|
| Supported glTF extensions | `m_supportedExtensions` in `src/gltf_scene.cpp` |
| Visualization / debug modes | `enum Visualization` in `shaders/shaderio.h` |
| Command-line parameters (names, ranges, defaults) | `parameterRegistry.add(...)` in `src/main.cpp`, `src/renderer*.cpp`, `src/benchmarking.cpp` |
| Menu labels, keyboard shortcuts, UI panels | menu/UI builders in `src/ui_renderer.cpp` (and other `src/ui_*`) |
| Material extension gates | `MAT_EXT_*` in `shaders/gltf_material_config.h`, `GLTF_USE_*` in `shaders/gltf_eval_config.h` |
| Host/device structs & layout | `shaders/shaderio.h` + the `*_io.h.slang` / `*_shaderio.h.slang` headers |
| Which files are tests/benchmarks | `tests/CMakeLists.txt` |
| CMake options & defaults | root `CMakeLists.txt` |

## Source layout (one line each; full map in developer.md)

- `src/renderer*.{cpp,hpp}` — render backends (`BaseRenderer` → `PathTracer` /
  `Rasterizer`), plus a separate `Silhouette` compute pass, all orchestrated by
  `renderer.cpp` (`GltfRenderer`).
- `src/gltf_scene*.{cpp,hpp}` — scene loading, GPU upload (`SceneVk`),
  acceleration structures (`SceneRtx`), editing, animation, merge, compaction.
- `src/ui_*` — ImGui panels (inspector, scene browser, animation); viewport and
  menus live in `ui_renderer.cpp`.
- `src/dlss*`, `src/optix_denoiser*`, `src/vk_cuda*` — AI denoisers + CUDA interop.
- `src/gizmo_*` — transform gizmo, grid, overlays.
- `shaders/*.slang` + `shaders/shaderio.h` — GPU code and host/device structs.

## Invariants — do not break these

- **The two render paths share resources.** Scene buffers, textures, and
  descriptors are built once and consumed by both. A change for one path must
  not fork or duplicate that shared state — if you touch shared resources,
  verify both paths still work.
- **RenderNode / RenderPrimitive arrays are derived and regenerated** from the
  glTF model (see RENDERING_ARCHITECTURE.md). Don't hand-mutate them as if they
  were source of truth; drive changes from the model / editor path so GPU SSBO
  and TLAS stay in sync.
- **The path tracer is the material reference.** New glTF material extensions
  land in the path tracer's Slang material eval first
  (`shaders/gltf_material_eval.h.slang`, gated by `MAT_EXT_*` flags in
  `shaders/gltf_material_config.h`). The rasterizer follows, it does not lead.
- **Host/device structs live in `shaders/shaderio.h`** (and the `*_io.h.slang` /
  `*_shaderio.h.slang` headers). Change both sides together; never let CPU and
  GPU layouts drift.
- **Scene editing is non-destructive** and fully undoable — route mutations
  through the editor + `undo_redo` system, not ad-hoc.

## Fix what you find — auto-fix on discovery

When you find a bug, drift, gap, or clear defect while working — **even if it's outside the scope

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nvpro-samples/vk_gltf_renderer](https://github.com/nvpro-samples/vk_gltf_renderer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
