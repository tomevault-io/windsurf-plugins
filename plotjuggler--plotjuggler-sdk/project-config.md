---
trigger: always_on
description: PlotJuggler SDK — C++20 foundation libraries that make up the PlotJuggler plugin SDK and host-side
---

# CLAUDE.md

## Project Overview

PlotJuggler SDK — C++20 foundation libraries that make up the PlotJuggler plugin SDK and host-side
plugin loading. **Read-only submodule** inside PJ4: consumed as-is; changes happen in this repo,
not in the PJ4 superproject. This file is the root navigation node for the whole submodule;
`pj_plugins/CLAUDE.md` adds module-specific guidance, while `pj_base` has no separate CLAUDE.md.

> The columnar storage engine (`pj_datastore`) used to live here. It now lives in the PlotJuggler
> application repo as a top-level module: plugins reach storage only through the C ABI defined in
> `pj_base` (the host-side write implementations are not part of the SDK), so the engine does not
> belong in the plugin SDK.

### Modules

- **pj_base** — vocabulary types (`Timestamp`, `DatasetId`, `Expected<T>`, `Span<T>`, type trees),
  the canonical builtin object vocabulary (`pj_base/builtin/`: 17 struct headers — Image, DepthImage,
  PointCloud, CompressedPointCloud, OccupancyGrid(+Update), Mesh3D, VideoFrame,
  SceneEntities, RobotDescription, CameraInfo, Log, ImageAnnotations, FrameTransforms, PosesInFrame,
  VoxelGrid, PlotMarkers) and their canonical wire codecs, the C-ABI protocol headers for
  DataSource/MessageParser/Toolbox + the C++ SDK base classes / host-view helpers built on them, the
  standalone C++17 functional parser-module authoring kit (`pj_base/parser_module/`), the host-side
  wasm parser-module manifest custom-section codec, and the test-only static WASI ABI auditor. The
  0.22 authoring helper builds native parser modules only; wasm loading/execution is not present.
- **pj_plugins** — host-side loaders + RAII handles + plugin **discovery** (directory scan +
  embedded-manifest inspection) for four plugin families (DataSource, MessageParser, Dialog, Toolbox),
  parser claim admission/resolution and native functional parser-module execution,
  config-envelope helpers, and the **dialog C ABI** (`pj_plugins/dialog_protocol/`). The
  duplicate-resolution *catalog* (which copy wins by priority/version/compatibility) is host policy
  and lives in the app (`pj_runtime`), built on these discovery primitives. Note the split: the DataSource/MessageParser/Toolbox C-ABI
  protocol headers live in `pj_base`; the **Dialog** protocol header lives here, not in `pj_base`.

### Dependency graph

- `pj_plugins` → `pj_base` (+ nlohmann/json)

## Read path

```text
this CLAUDE.md -> relevant docs -> headers -> code
```

Start here to pick the module + source-of-truth doc. Read the doc before treating code as
authoritative for *intent*: code shows current implementation; docs define intended architecture,
public contracts, terminology, and module boundaries. If docs and code disagree, that is a
documentation bug — do not silently let stale docs survive. Any change to behavior, public APIs,
ABI structs, SDK types, module ownership, plugin workflows, or storage formats must include a
documentation check before commit.

## Key Documentation

**Project-wide** (`docs/`):

| Document | Content |
|----------|---------|
| `docs/builtin_type.md` | Canonical builtin object types — the shim between third-party schemas and PJ internals; lists every builtin + its codec |
| `docs/image_annotations_format.md` | Canonical `PJ.ImageAnnotations` wire format |
| `docs/dialog-sdk-reference.md` | Quick reference for `WidgetData` setters + `DialogPluginTyped` event handlers |
| `docs/cpp_design_recommendations.md` | C++ style, error handling, API design guidelines |
| `docs/toolbox-porting-gap-analysis.md` | Historical PJ3→PJ4 toolbox SDK gap analysis (most gaps now closed; read as context, not current reference) |
| `V4_STORE.md` | ObjectStore plugin ABI: services, ownership rules, lazy fetch |

**Plugin system** (`pj_plugins/docs/`): `REQUIREMENTS.md` (families, capability system, config
contract) · `ARCHITECTURE.md` (C ABI protocols, SDK base classes, host loaders, dialog protocol) ·
`data-source-guide.md` · `message-parser-guide.md` · `dialog-plugin-guide.md` · `toolbox-guide.md`.
The concise native functional-module authoring reference is
`.claude/skills/plotjuggler-plugin/references/parser-module.md`.

## Build & Test

```bash
./build.sh            # RelWithDebInfo (build/)
./build.sh --debug    # Debug + ASAN (build/debug_asan)
./test.sh             # runs tests in all discovered build dirs
```

Dependencies come from Conan (`conanfile.py`). Before committing always run
`./build.sh --debug && ./test.sh`. Formatting/linting (clang-format, pinned to v22.1.0 in `.pre-commit-config.yaml`) is enforced by
pre-commit hooks. Verify docs match reality before any commit that changes behavior, public APIs, ABI structs,
SDK types, or storage formats; if stale and not asked to update, ask before committing.

## Release Versioning

The version is a **plugin-compatibility contract** (plugins pin this SDK by Conan range), not
decoration. In every PR, proactively raise whether a release is warranted and propose the bump.
The bump is decided by **plugin impact**, semver-style:

- **MAJOR** (`X.0.0`) — an **ABI or API break**: an existing plugin must be recompiled or its
  source changed to keep working. Removing/reordering ABI vtable slots, changing a struct layout or

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PlotJuggler/plotjuggler_sdk](https://github.com/PlotJuggler/plotjuggler_sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
