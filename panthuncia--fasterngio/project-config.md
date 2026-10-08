---
trigger: always_on
description: FasterNGIO generates Skyrim SE grass caches in NGIO's `.cgid` format, with NGIO-style "no grass in
---

# AGENTS.md

FasterNGIO generates Skyrim SE grass caches in NGIO's `.cgid` format, with NGIO-style "no grass in
objects" rejection ray traced on the GPU (D3D12 or Vulkan) or, as a fallback, against a CPU BVH. It is
a standalone, GPL-3 command-line tool for Windows and Linux: it must not depend on SARP, BasicRenderer
or CommonLibSSE.

## Layout

| Path | Contents |
|---|---|
| `src/GameData/` | Plugin (ESM/ESP/ESL) reading, in the order data flows: `LoadOrder` (plugins.txt, TES4 headers, FileIDs), `PluginParser` (one plugin's records into a `StaticPluginShard`), `StaticWorld` (overrides resolved into the `StaticWorldSnapshot`). `FormID.h` and `Records.h` hold the types, `Internal/` the file format (`RecordReader`) and the per-record extractors. Only the fields placement and rejection use are read. |
| `src/Grass/` | Grass placement: `Placement.h` (settings, candidates, `GenerateCellCandidates`), `VanillaPlacement.cpp` (the engine's algorithm), `SmoothPlacement` (`SmoothWeightField` and smooth placement), `Internal/PlacementCommon.h` (terrain sampling, filters and blade encoding both share). `CellCache` turns placed blades into a cell's `.cgid` (`FinalizeCell`: the engine's blocks and quadrant cap; file names); `GrassModels` measures each GRAS model's blades per block; `NgioCacheWriter` is the format; `GameIni` the game's `[Grass]` INI settings; `LandTexture` which land textures the terrain shows where (NGIO's texture forms). |
| `src/Archives/` | Memory-mapped BSA (v103/104/105) reader and load-order-aware resolver (loose files win). |
| `src/Platform/` | `DataDirectory`: case-insensitive, either-separator resolution of Data paths (an index off Windows); `IniFile`; `GameInstall` (validates a game folder, detects the store build, derives plugins.txt and the INI folder); `ModOrganizer` (the MO2 instance and profile this process runs under); `UserSettings` (the launcher's remembered choices); the small shared helpers: `FileSystem` (per-user folders), `Text` (UTF-8 paths, ASCII case folding), `MappedFile`, `WholeFile`. |
| `src/Collision/` | nifly-based Havok collision extraction (CMS, packed strips, convex hulls, boxes, spheres, capsules) into model space; on request, the render shapes' names and vertices (`NearestRenderShape`, for NGIO's shape-name filters), or (experimental `renderGeometry`) the visible render triangles in place of a model's collision. `InstanceShape`: the triangle and vertex counts of the shape the engine instances grass from. |
| `src/Rejection/` | NGIO query shapes (`RejectionConfig`), the per-world instance index (`WorldIndex`), the CPU BVH fallback (`CpuBvh`), the brute-force reference (`CpuReference`), and `Bounds` (AABBs and primitive kinds, shared with the GPU). `RejectionFeatures` holds NGIO's resolved lists and objects; `NgioRules` the per-blade decision (ignored shapes, grass cliffs) that every backend feeds. |
| `src/Seasons/` | Seasons of Skyrim's swap types and the automatic winter swaps (`SeasonSwaps`). |
| `src/Gpu/` | `GpuRejector`: feature check, the render thread (OpenRenderGraph `PersistentGraphHost`), BLAS/TLAS, ray-tracing pipeline, one DispatchRays per frame, readback. `ModelPacking` lays collision out for the shader (in Core, so it is testable without a GPU); `GpuShaders` loads the shader library. |
| `src/Pipeline/` | Lock-free cell pipeline on ORGModuleServices' AsyncStateGraph (`CellPipeline`), the writer threads (`FileWriterPool`), the waiters that suspend in the graph (`SuspensionWaiters`), the TBB graph scheduler. |
| `src/Concurrency/` | Header-only lock-free building blocks: `MpscQueue`, `WaitUntil`. |
| `apps/FasterNGIO/` | The `FasterNGIOApp` library: `CommandLine` (arguments, MO2 defaults), `Options` (`GenerateOptions` and the names of its choices), `Generate` (the run itself: snapshot, per-world loop, progress and cancellation), `WorldSetup` (preparing a world), `Diagnostics`, `NgioConfig` (NGIO's `GrassControl.ini` and `*_NGIO.ini`), `SeasonsConfig` (Seasons of Skyrim's settings, swap files and per-world passes). The executable adds `Main.cpp` (CLI or launcher) and `Gui/` (the ImGui launcher; Win32 + D3D11 on Windows, GLFW + OpenGL 3 elsewhere). |
| `shaders/` | `GrassRejection.hlsl` (DXIL for D3D12, SPIR-V for Vulkan), `Shared/GrassQueryMath.hlsli` (the overlap tests and segment hits) and `Shared/RejectionLayout.hlsli` (the buffer and root-constant layouts), which the C++ also compiles (`src/Rejection/HlslShim.h`). |
| `tests/` | One file per module; `TestSupport.h` has the temp-folder and file helpers. `GameDataTests` writes tiny plugins and archives; `RejectionTests` builds synthetic worlds. |
| `tools/` | `build_linux.sh` and `setup_linux.sh` (sharing `linux_common.sh`); `analyze_placement_edges.py`: grid-locking and corner statistics of `--export-blades` output, with a grid-free control. |
| `external/` | Submodules: BasicRHI, OpenRenderGraph, ORGModuleServices, BasicTelemetry, volk (pinned to BasicRenderer's commits) and upstream nifly. |

## Build and test

```powershell
cmake --preset vs2026          # GPU build (D3D12 + Vulkan); vs2026-cpu builds without the GPU stack
cmake --build --preset vs2026
ctest --preset vs2026
```

```sh

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [panthuncia/FasterNGIO](https://github.com/panthuncia/FasterNGIO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
