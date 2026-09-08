---
trigger: always_on
description: Cytnx is a C++ tensor-network library with CUDA acceleration and pybind11 Python
---

# Cytnx — guide for coding agents

Cytnx is a C++ tensor-network library with CUDA acceleration and pybind11 Python
bindings. This file is the canonical operational contract for anyone — human or
agent — working in Cytnx. For *what* to work on, see issue #759; this file is
only about working *correctly*.

## Skills

Task-specific playbooks live in [`.agents/skills/`](.agents/skills/), one
directory per skill, in the interoperable Agent Skills layout that Codex, Gemini
CLI, and Claude Code all discover. Use them instead of re-deriving commands or
rules:

- [`build-test-workflow`](.agents/skills/build-test-workflow/SKILL.md) — before
  any build or test run (any preset, C++ or Python, with or without a GPU).
- [`cross-revision-benchmark`](.agents/skills/cross-revision-benchmark/SKILL.md)
  — before making or checking any performance claim.
- [`google-cpp-style`](.agents/skills/google-cpp-style/SKILL.md) — writing or
  reviewing C++/CUDA (`.cpp`/`.hpp`/`.cu`).
- [`google-python-style`](.agents/skills/google-python-style/SKILL.md) — writing
  or reviewing Python (`cytnx/`, `pytests/`, `tools/`).

## Build

Configure with a preset (see `CMakePresets.json`), then build. Artifacts land in
`build/<preset>/`.

```bash
git submodule update --init --recursive          # first time (hptt, morse_cmake), else configure fails
cmake --preset openblas-cpu                       # or: mkl-cpu, openblas-cuda, mkl-cuda, openblas-apple
cmake --build --preset openblas-cpu
```

Key toggles (set by presets): `USE_CUDA`, `USE_MKL`, `BUILD_PYTHON` (ON by
default), `USE_HPTT`. CPU presets iterate fastest; only use a `*-cuda` preset
when a change actually touches GPU code — but when it does, build the cuda preset
to compile-check it **even without a GPU** (the CUDA toolkit installs on GPU-less
machines, including cloud agents; only *running* GPU tests needs real hardware).

## Test

C++ tests need `RUN_TESTS=ON` (a `debug-*` preset sets this already). The full
build/test/benchmark mechanics live in the `build-test-workflow` and
`cross-revision-benchmark` skills — take the commands from there.

## Format — must match CI exactly

- clang-format is **pinned to v14** (`.clang-format`: Google base, column 100,
  C++20). A newer clang-format reflows differently and **fails CI** — do not
  hand-format, and do not use a different version.
- Run the hooks instead of formatting by hand:
  ```bash
  pre-commit run --all-files          # or: pre-commit run --files <changed>
  ```
- The clang-format hook rewrites files in place and **aborts the commit** if it
  changed anything → `git add` the reformatted files and commit again.

## Coding style

New code follows the **Google style guides** — the project baseline
(`.clang-format` is `BasedOnStyle: Google`). The `google-cpp-style` and
`google-python-style` skills carry the full rules plus the Cytnx-specific
overrides; read them when writing or reviewing code.

The rules agents most often get wrong:

- **No leading underscore in new C++ code** (strict). `_foo` is the retired
  private-member convention (#836); new C++/CUDA members, helpers, and locals
  must not start with `_`. **The ban is C++-only.** In Python a single leading
  underscore is Google style's marker for a module- or class-internal name
  (`_helper`) and is correct there — see `google-python-style`.
- **snake_case for new identifiers**; type names stay `PascalCase`. Existing code
  is mixed — match a file's convention when it is consistent, don't propagate
  messy naming, and don't reformat old files as churn.
- **Trailing underscore = in-place mutator returning `*this`** (`Add_`,
  `contiguous_`) — a Cytnx convention, *not* Google's data-member meaning.
- **Pass scalars by value, not `const&`.** Built-ins, enums, `Scalar`, and
  `complex<double>` are never faster by reference in Cytnx (a value rides in
  registers; `const&` forces a stack spill + indirection). Reserve `const&` for
  large objects (`Tensor`, `Storage`, `UniTensor`, containers).
- **C++20 / CUDA 20 are the standard** (`CMAKE_CXX_STANDARD` and
  `CMAKE_CUDA_STANDARD` are both `20`). Concepts, `<bit>`, designated
  initializers, `constexpr` algorithms, and `cuda::std::*` are all fair game;
  never add C++17-compat workarounds or `#if __cplusplus` guards.
- **Complex scalars in device code are `cuda::std::complex<T>`** — not
  `cuComplex` / `cuDoubleComplex`, not host `std::complex`, not
  `thrust::complex`.

### Robust C++ in touched code

Treat surrounding Cytnx code as historical context, not as an automatic style
guide. When writing or modifying code:

- Pass cheap scalar values by value, not by `const&`. This includes `bool`,
  integral types, enum-like dtype/device values, floating-point values, `Scalar`,
  and complex scalar types.
- Avoid `cytnx_XXXX` typedefs for ordinary local programming types. Use standard
  C++ types for counters, sizes, flags, loop indices, tolerances, and local
  arithmetic unless the value is specifically a dtype-backed tensor/storage
  scalar.
- Do not add magic dtype integers. Use named dtype constants, type traits, or
  established dispatch helpers.
- Do not copy raw-memory idioms such as `&vec[0]`, vector-plus-`memcpy`, manual
  container copies, or untyped `void*` handling unless the code is genuinely

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cytnx-dev/Cytnx](https://github.com/Cytnx-dev/Cytnx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
