---
trigger: always_on
description: <!-- SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved. -->
---

<!-- SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved. -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

# AGENTS.md

Entry point for any coding agent (Claude Code, Codex/GPT, etc.) working in this repository. `CLAUDE.md` is a symlink to this file so Claude Code reads it automatically; non-Claude agents read `AGENTS.md` directly. Either way, this is the single source of truth.

## Project Overview

Scene Optimizer is a standalone C++ library providing 45+ USD scene optimization operations (geometry, materials, hierarchy, analysis) with a plugin system and Python bindings.

## Build & Test Commands

The repo ships two equivalent entry scripts: `./repo.sh <command>` for Linux/bash-compatible shells, and `repo.bat <command>` for Windows `cmd.exe`/PowerShell. They accept the same arguments — pick whichever matches the active shell.

```bash
# Build
./repo.sh build      # Linux / bash
repo.bat build       # Windows cmd.exe / PowerShell

# Run all tests (cpp, python)
./repo.sh test
repo.bat test

# Check code format (CI)
./repo.sh ci format
repo.bat ci format
```

## Code Style

- **C++:** Allman braces, 4-space indent, 120 column limit (`.clang-format`)
- **Python:** Flake8, 120 column limit, max complexity 18 (`.flake8`)
- **C++ standard:** C++17

## Architecture

### Plugin System

The core is a C++ plugin architecture. Each optimization operation is a shared library that subclasses `omni::scene::optimizer::Operation` (in `source/core/src/Operation.h`) and registers itself with `SO_PLUGIN_INIT`. Plugins in `source/operations/` are auto-discovered at build time via premake.

### Key Layers

1. **Public API** — `include/omni/sceneoptimizer/ISceneOptimizer.h` — C++ interface consumed by external callers
2. **Core Library** (`source/core/`) — Operation manager, Python bindings via pybind11 (`source/core/bindings/BindingsPython.cpp`)
3. **Operations** (`source/operations/`) — 45+ plugins; each is a standalone `.cpp` file (and optional headers) that registers one operation

### Operation Lifecycle

Arguments are declared in the constructor via `addArgument()`; their bound member variables are auto-populated from JSON/UI before `executeImpl()` is called. After execution, arguments reset to defaults. Access the USD stage via `getUsdStage()`.

### Test Organization

```text
source/tests/
├── test.cpp/omni.scene.optimizer.core/   # C++ unit tests (Doctest)
├── test.python/                          # Python binding tests
└── test.cuda.utils/                      # Helper shared lib for the cpp suite (not its own runnable suite)
```

## Writing a New Operation

1. Create a `.cpp` (and optional `.h`) under `source/operations/`
2. Subclass `omni::scene::optimizer::Operation`; call base constructor with `(key, displayName, description)`
3. Declare arguments in the constructor with `addArgument()`
4. Implement `executeImpl()` returning `OperationResult::eSuccess`
5. Register at the bottom: `SO_PLUGIN_INIT(omni::scene::optimizer::MyPlugin);`
6. To support analysis mode, override `getSupportsAnalysis()` → `true` and implement `executeAnalysisImpl()`
7. Add docs entry in `operations.rst`

Display group constants: `s_displayGroupGeometry`, `s_displayGroupMaterials`, `s_displayGroupStage`, `s_displayGroupUtilities`

See `PLUGINS.md` for the full plugin authoring guide and `source/core/src/Argument.h` for all display types and argument configuration methods.

## Code Coverage (Linux only)

Coverage tooling is Linux-only (gcov), so use `./repo.sh` only:

```bash
./repo.sh build --rebuild --config "release" --enable-gcov
./repo.sh test
./repo.sh cxx_coverage --collect --generate-html --remove --report
# Output: ./_build/coverage/
./repo.sh cxx_coverage --zero-coverage  # Reset counters
```

## Performance Validators

Scene Optimizer ships an `omniverse-asset-validator` integration: a set of rules under the `Performance` category that wrap analysis-mode operations. Most register by default; a few are opt-in because they're slow on large stages — see `_default_rule_classes()` and `_expensive_rule_classes()` in `source/core/python/omni/scene/optimizer/validators/_plugin.py` for the authoritative lists. Tests at `source/tests/test.python/test_validators_*.py`.

Two perf-relevant features in `_base.py`: an analysis-result cache keyed by root-layer identifier (so e.g. the mesh-cleanup-family rules share one `meshCleanup` analysis), and a `REQUIRES_MESH` short-circuit that skips mesh-only rules on stages without `UsdGeomMesh` prims.

Use the `validators` skill (`.agents/skills/validators/SKILL.md`) for invocation, file logging, and adding new validators. The asset-validator discovers plugins via `importlib.metadata` entry points, so **`omni_asset_validate` only sees Scene Optimizer after the `omniverse-scene-optimizer` wheel is pip-installed** (source-tree `PYTHONPATH` alone registers no entry point metadata). Third-party callers must either invoke `register_all()` programmatically (use `include_expensive=True` to include `FindOverlappingMeshes`) or pip-install the wheel and set `OMNI_ASSET_VALIDATOR_ISOLATE_ENTRYPOINTS` for CLI allow-listing.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-Omniverse/usd-optimize](https://github.com/NVIDIA-Omniverse/usd-optimize) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
