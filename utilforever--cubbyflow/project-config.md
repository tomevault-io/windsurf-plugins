---
trigger: always_on
description: Guidance for AI coding agents working in this repository. Humans should start with [README.md](README.md); this file puts project-specific rules in the order an agent usually needs them.
---

# AGENTS.md

Guidance for AI coding agents working in this repository. Humans should start with [README.md](README.md); this file puts project-specific rules in the order an agent usually needs them.

## What this repository is

CubbyFlow is a voxel-based fluid simulation engine with C++23 host code and optional C++17 CUDA device code. It is based on the Jet framework and contains:

- CPU implementations of math, geometry, grids, particles, spatial search, level sets, pressure solvers, and fluid solvers.
- Matching 2-D and 3-D APIs for most simulation domains.
- An optional CUDA backend under `Includes/Core/CUDA/` and `Sources/Core/CUDA/`.
- A Python module named `pyCubbyFlow`, exposed through pybind11.
- C++, CUDA, Python, manual, memory-performance, and time-performance tests.
- Standalone simulation and conversion examples under `Examples/`.

The public C++ API lives under `Includes/Core/`; implementations live under `Sources/Core/`. Start there for almost every behavior change.

## Golden rules

1. **Keep 2-D and 3-D behavior aligned.** Most core algorithms, data structures, aliases, explicit template instantiations, bindings, and tests have dimensional counterparts. Check the sibling implementation before changing one side. Some geometry is inherently 3-D, so follow the existing domain rather than creating a meaningless counterpart.
2. **Keep the public API and Python-visible behavior in sync.** When changing a public type, method, default, enum, or solver behavior, inspect `Includes/API/Python/`, `Sources/API/Python/`, `Sources/API/Python/main.cpp`, and `Tests/PythonTests/`.
3. **Fix shared behavior at the shared layer.** Use `rg` to find every caller, override, binding, test, and dimensional specialization before editing. Avoid one-off guards in callers when the invariant belongs in a common base class or utility.
4. **Use existing patterns before adding code.** Search the neighboring domain for templates, builders, aliases, numerical helpers, parallel loops, serialization code, and tests. Do not add a new abstraction or dependency when the repository already has the required shape.
5. **Use CMake targets as the source of truth.** Read the root and nearest `CMakeLists.txt` before adding files, dependencies, compile definitions, or platform-specific behavior. Reconfigure CMake after adding source files; several targets use `GLOB` or `GLOB_RECURSE` without automatic reconfigure.
6. **Preserve C++23 portability.** CI builds with GCC, Clang, and MSVC across Linux, macOS, and Windows. Avoid compiler extensions unless they are isolated behind existing CMake checks.
7. **Treat warnings as failures.** `CUBBYFLOW_WARNINGS_AS_ERRORS` defaults to `ON`. Fix warnings in project code instead of suppressing them globally.
8. **Keep CUDA optional.** CPU-only configurations must continue to work. CUDA-only code belongs in the existing CUDA directories and should not leak into ordinary builds without guards. Host code uses C++23; CUDA device code remains C++17 for the supported toolchains.
9. **Do not hand-edit generated files alone.** FlatBuffers schemas live in `Sources/Core/Flatbuffers/schema/` and checked-in generated headers live in `Sources/Core/Flatbuffers/generated/`. A schema change must update both; use the FlatBuffers version constrained by `vcpkg.json`.
10. **Run the smallest relevant check.** Documentation-only changes need only document validation. Behavior changes need focused C++, Python, or CUDA coverage before broader validation.

## How the core code is shaped

### Dimensional templates

Many public types use `template <size_t N>` and expose readable aliases such as `Sphere2` and `Sphere3`. Their `.cpp` files commonly end with explicit instantiations:

```cpp
template class Sphere<2>;
template class Sphere<3>;
```

When extending this pattern:

- Put dimension-independent logic in the shared template.
- Preserve `Foo2`/`Foo3` and pointer aliases when the public type exposes them.
- Add both explicit instantiations when both dimensions are supported.
- Update both `Foo2Tests.cpp` and `Foo3Tests.cpp`, or the existing combined test file, when behavior applies to both.
- Check Python registration for matching `AddFoo2(m)` and `AddFoo3(m)` calls.

### Public headers and implementations

- Use project includes such as `<Core/Geometry/Sphere.hpp>`.
- Keep code in the `CubbyFlow` namespace.
- Public declarations and Doxygen comments belong in `Includes/Core/`.
- Non-inline implementation belongs in the matching `Sources/Core/` domain.
- Template implementation headers use the existing `-Impl.hpp` convention when definitions must be visible to callers.
- Follow nearby ownership aliases and builder APIs. If a type already provides `GetBuilder()`, `Build()`, and `MakeShared()`, extend that builder instead of adding a second construction mechanism.

### Python bindings

Python binding declarations mirror the core domains under `Includes/API/Python/`; definitions mirror them under `Sources/API/Python/`. Every exposed binding is registered in `Sources/API/Python/main.cpp`.

For a new or changed Python-visible API:

1. Update the binding declaration and implementation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [utilForever/CubbyFlow](https://github.com/utilForever/CubbyFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
