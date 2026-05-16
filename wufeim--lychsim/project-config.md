---
trigger: always_on
description: This file is the contributor and AI-agent guide for the LychSim repository. **Both human contributors and AI assistants** (Claude, Claude Code, and similar agents) working on this codebase or its forks should treat the conventions below as authoritative -- they keep the public API, documentation, and tests aligned across changes.
---

# CLAUDE.md

This file is the contributor and AI-agent guide for the LychSim repository. **Both human contributors and AI assistants** (Claude, Claude Code, and similar agents) working on this codebase or its forks should treat the conventions below as authoritative -- they keep the public API, documentation, and tests aligned across changes.

For the project pitch and example usage, see [README.md](README.md). For the v1.0.0 feature surface and planned future work, see [ROADMAP.md](ROADMAP.md).

## Quick Start for Contributors

```bash
git clone https://github.com/wufeim/LychSim.git
cd LychSim
pip install -e .[docs]              # package + docs extras (Sphinx, theme, sphinx-design)
```

Most code paths assume a running LychSim / UnrealCV instance. Quickest way: download a pre-built scene binary and launch it via [`lychsim run`](https://wufeim.github.io/LychSim/docs/cli.html) -- see the [Quick Start tutorial](https://wufeim.github.io/LychSim/tutorials/quick_start.html). For UE5 setup from source, see the [installation guide](https://wufeim.github.io/LychSim/tutorials/installation.html).

```bash
pytest tests/                                 # unit + integration tests (need a running UE)
black src/ && flake8 src/                     # format + lint
cd docs && make html                          # build the docs to docs/_build/html
```

## Architecture

LychSim is layered:

- **Communication layer** -- [`api/client.py`](src/lychsim/api/client.py): low-level socket client implementing the UnrealCV binary protocol. Thread-safe request/response.
- **Mid-level API** -- [`api/api.py`](src/lychsim/api/api.py): `UnrealCv_API` maps Python methods to raw text commands.
- **High-level wrapper** -- [`api/wrapper/`](src/lychsim/api/wrapper/): `LychSim`, the user-facing class, composed from three mixins:
  - [`CameraCommandsMixin`](src/lychsim/api/wrapper/camera_mixin.py) -- image capture (RGB / depth / normal / segmentation / point map / z-buffer), pose, intrinsics.
  - [`ObjectCommandsMixin`](src/lychsim/api/wrapper/object_mixin.py) -- list / spawn / delete / query / update actors, bounding boxes, per-object annotations.
  - [`DataCommandsMixin`](src/lychsim/api/wrapper/data_mixin.py) -- pause / resume, debug-line drawing.
- **Core data structures** -- [`core/`](src/lychsim/core/): `Object`, `AABB`, `OBB`, `SemanticScene` / `SemanticLevel` / `SemanticRegion`. Serializable to dict and `.npz`.
- **CLI** -- [`cli/`](src/lychsim/cli/): the `lychsim` command. See the [CLI reference](docs/docs/cli.rst) for every subcommand.
- **MCP server** -- [`mcp/`](src/lychsim/mcp/): a thin FastMCP layer exposing the wrapper to LLM clients (Claude Code, Claude Desktop) over stdio. One tool module per mixin under [`mcp/tools/`](src/lychsim/mcp/tools/).
- **Scripts** -- [`scripts/`](scripts/): standalone utilities that connect to a running LychSim instance.

The high-level wrapper is the canonical API. Contributor code should add methods to the appropriate mixin rather than calling `client.request(...)` directly.

## Coordinate System & Geometry Conventions

Unreal Engine (and therefore LychSim) uses a **left-handed, Z-up** coordinate system. All positions are in **centimeters**, rotations in **degrees** as `[pitch, yaw, roll]`.

### Bounding box flavors

The C++ plugin exposes four bbox shapes per object via `get_obj_aabb` / `get_obj_obb` / `get_obj_annots`:

- **`aabb`** -- `FActorController::GetAxisAlignedBoundingBox()`. Typically reflects the root / collision component only and understates Blueprint composites.
- **`bounds`** -- `Actor->GetActorBounds(false, ...)`. Aggregates *all* registered primitive components, including non-colliding visual meshes. **Use this for visual alignment** (spawn offsets, stacking).
- **`bounds_tight`** -- `Actor->GetActorBounds(true, ...)`. Only colliding components.
- **`obb`** (Python `get_obj_obb`) -- same data as `bounds`, returned as flat `center extent rotation`.

When in doubt, prefer `bounds` / `get_obj_obb` over `aabb`.

### Spawn offset convention

Assets have their pivots at arbitrary positions. To place an object so its visual bbox bottom-center lands on a desired world location:

1. Pre-compute the offset by spawning the asset at origin with rotation 0, querying `bounds`: `offset = (-cx, -cy, ez - cz)`.
2. At spawn time: `location = desired_loc + offset`, `yaw = desired_yaw + annotated_calibration_rotation`.

The pre-computed `mesh_offset_{x,y,z}` and `mesh_extent_{x,y,z}` columns ship with the [`wufeim/lychsim_objects`](https://huggingface.co/datasets/wufeim/lychsim_objects) Hugging Face dataset.

## Code Conventions

- **Python ≥3.10** required -- the wrapper uses PEP 604 union syntax (`X | Y`) at function-definition time without `from __future__ import annotations`.
- **`numpy==1.26.4` is pinned**. Don't change this version without a coordinated upgrade of the rest of the package.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wufeim/LychSim](https://github.com/wufeim/LychSim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
