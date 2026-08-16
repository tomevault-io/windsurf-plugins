---
trigger: always_on
description: Momentum is a C++ library for human kinematic motion, character representation,
---

# Momentum AI Assistant Guide

Momentum is a C++ library for human kinematic motion, character representation,
and numerical optimization solvers. The repository also contains PyMomentum,
the Python bindings and Python utilities built on top of the C++ library.

This guide is for AI coding agents working in the public repository. Keep future
instructions codebase-focused and self-contained: use public paths, public
commands, and public documentation. Do not add private organization-specific
workflow, review, source-control, or infrastructure rules here.

For end-user installation, building from source, and the contribution workflow,
see `README.md` and `CONTRIBUTING.md`.

## Repository Map

- `momentum/` - C++ core library.
  - `common/` - exceptions, checks, logging, shared utilities.
  - `math/` - transforms, meshes, random utilities, numerical helpers.
  - `character/` - skeletons, characters, skinning, collision geometry.
  - `solver/` - generic solver framework.
  - `character_solver/` - character-specific inverse kinematics solvers.
  - `character_solver_simd/` - SIMD-accelerated solver error functions.
  - `character_sequence_solver/` - multi-frame sequence solvers.
  - `diff_ik/` - differentiable inverse kinematics support.
  - `simd/` - SIMD helpers and abstractions shared by the compute paths.
  - `io/` - file I/O including FBX, glTF, URDF, C3D, and related formats.
  - `marker_tracking/`, `rasterizer/`, `camera/`, `gui/` - application-facing
    support libraries.
  - `test/` - C++ unit tests and test helpers.
  - `examples/`, `tutorials/` - example applications and tutorials.
  - `website/` - documentation site sources.
- `pymomentum/` - Python bindings and Python modules.
  - `geometry/` - NumPy-oriented character and geometry bindings.
  - `diff_geometry/`, `solver/`, `tensor_momentum/` - PyTorch-backed modules.
  - `solver2/`, `marker_tracking/`, `renderer/`, `camera/`, `axel/` - Python
    extension modules and helpers.
  - `test/` - Python tests.
- `axel/` - bundled Axel acceleration library (sources under `axel/axel/`).
- `cmake/` - CMake helper functions and source lists. `cmake/mt_defs.cmake`
  defines the `mt_library`, `mt_test`, and `mt_executable` helpers used
  throughout `CMakeLists.txt`.
- `scripts/` - wheel, packaging, and CI helper scripts.
- `.github/workflows/` - GitHub Actions CI, nightly, wheel, and docs workflows.
- `CMakeLists.txt`, `pixi.toml`, `pyproject.toml` - top-level build, task, and
  Python packaging entry points.

## Build And Test

Use Pixi tasks for normal development. Dependencies install automatically on the
first run.

```bash
pixi run build          # Release C++ build, with Python bindings enabled
pixi run build_dev      # Debug C++ build
pixi run test           # Release C++ tests
pixi run test_dev       # Debug C++ tests
pixi run test_verbose   # Release C++ tests with verbose output
pixi run build_py       # Build Python bindings
pixi run test_py        # Python tests
pixi run lint           # Format C++ code with clang-format
pixi run lint-check     # Check C++ formatting
pixi task list          # Show all available tasks
```

Useful environment-specific commands:

```bash
pixi run -e py313 build_py
pixi run -e py313 test_py
pixi run -e gpu build_py
```

Windows developers can use `pixi run open_vs` to open the configured Visual
Studio project.

## Platform Notes

- macOS builds are supported on both Intel and Apple Silicon.
- On Linux the FBX SDK is downloaded automatically during environment setup
  (the `install_fbxsdk` task).
- FBX I/O is available on Linux, macOS, and Windows (x86_64); USD I/O requires
  the optional USD dependencies.
- SIMD-accelerated paths target x86_64 (AVX2/FMA); other architectures such as
  ARM fall back to scalar implementations.

## Source Lists And Generated Files

- When adding, removing, or renaming C++ files under `momentum/`, update
  `cmake/build_variables.bzl`. CMake reads these lists through
  `cmake/mt_defs.cmake`.
- When adding, removing, or renaming PyMomentum C++ binding files, update
  `pymomentum/cmake/build_variables.bzl`.
- When adding new public C++ classes that need forward declarations, update
  `momentum/gen_fwd_input.toml`. The per-module `fwd.h` files are generated from
  it — each class gets `_p`/`_u`/`_w` (and `_const_*`) smart-pointer aliases. Do
  not hand-edit `fwd.h`; change the TOML source instead.
- When changing PyPI packaging, edit `pyproject-pypi.toml.j2` and regenerate the
  derived files with `pixi run generate_pyproject`.
- Do not hand-edit generated build or packaging outputs when a source template
  exists.

## C++ Conventions

- Follow the C++ style guide in the developer guide on the documentation site:
  https://facebookresearch.github.io/momentum/docs_cpp/developer_guide/style_guide
- Use Doxygen-style `///` comments in public headers. Use `@param` and `@return`
  tags, and document new public classes, structs, and functions.
- Use `camelCase` for functions, variables, and members; `PascalCase` for class
  names; `ALL_CAPS` for macros; `snake_case` for files and directories; and a
  trailing underscore for private members.
- Write abbreviations in `PascalCase` too, e.g. `Fbx` and `Gltf`, not `FBX` or
  `GLTF`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [facebookresearch/momentum](https://github.com/facebookresearch/momentum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
