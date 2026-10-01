---
trigger: always_on
description: A C++23 computational geometry and mesh processing library
---

# Mesh-processing-library

A C++23 computational geometry and mesh processing library
(github.com/hhoppe/Mesh-processing-library).

## Build

Two parallel build systems cover the same sources:

- GNU make, with the machinery under `make/`.
- MSBuild, with roughly 24 `.vcxproj` files in a `.sln`, sharing `hhmain.props`
  and `hhmain_first.props`.

The make build has five configurations, selected with `CONFIG=`:

| CONFIG   | Compiler and standard library                |
| -------- | -------------------------------------------- |
| `unix`   | WSL/Linux Clang + libstdc++                  |
| `win`    | MSVC                                         |
| `clang`  | Windows Clang + MSVC STL                     |
| `mingw`  | Windows GCC + libstdc++                      |
| `cygwin` | Cygwin GCC + libstdc++ (`CC=clang` optional) |

`make -j12` builds all programs into `bin/<CONFIG>/` and runs the unit tests. Name a target
to build less, e.g. `make -j12 Filtermesh` or `make CONFIG=mingw -j12 libHh`.

- The default `CONFIG` is `win` on Windows and `unix` elsewhere.
- `CONFIG=win` defaults to a debug build (`release=0`); the others default to release.
- Under `unix`, the compiler is Clang by default; `CC=gcc` switches to GCC.
- `make CONFIG=all` (or `make makeall`) runs every configuration in turn.
- `PEDANTIC=1` enables the full warning set; adding `ignore_compile_warnings=0` also makes warnings
  errors (`-Werror`, or `-WX` under MSVC).
- Sanitizers work under `CONFIG=unix` only (mingw ships no sanitizer runtime):
  `make CONFIG=unix release=0 PEDANTIC=1 sanitize=address,undefined -j12 test`, or
  `sanitize=thread`.
- The Windows configurations run make under Cygwin. From WSL, launch them with
  `/mnt/c/cygwin64/bin/bash.exe -lc 'cd /hh/git/mesh_processing && make CONFIG=win -j12' </dev/null`.
- Toolchain paths can be overridden in `Makefile_local_defs` at the repository root.
- MSBuild places its executables in `bin/msbuild/` (`ReleaseMD - x64`) and `bin/msbuild_debug/`.
  `bin/` itself holds only tracked scripts, such as `mesh_to_pm`, `pm_simplify`, and `hcheck`.

Warnings are signal, not noise. Treat every new warning as a defect to fix, in every
configuration, not just the one you happened to build.

Toolchain discovery in the makefiles uses `$(wildcard)` rather than `$(shell)`, because
the recursive make invocations make shell subprocesses expensive. Keep it that way.

## Layout

- `libHh` is the core library (containers, geometry, meshes, images, video, audio);
  `libHh/README.md` gives an overview of its core classes.
- `libHwWindows` (Win32) and `libHwX` (X11) implement windowing; a build links one of them.
- Each program (`Filtermesh`, `MeshSimplify`, `G3dOGL`, ...) lives in its own directory and
  links against these libraries. `G3dVec` compiles sources from `G3dOGL`, and `Filtervideo`
  compiles `VideoViewer/GradientDomainLoop.cpp`, hence their ordering in the top-level
  `Makefile`.
- `test/` holds the unit tests. `make demos` builds all programs and runs `demos/`, which reads
  its inputs from `demos/data/` and writes all generated files into `demos/results/`.
  `DEMOS_HIDDEN=1 make -C demos view` runs the viewing demos non-interactively: no window is
  mapped and each viewer quits after a few seconds, which detects crashes, assertion failures,
  and sanitizer reports, but compares no rendered images.

## Test

- Run all unit tests with `make -j12 test`, or a single one with
  `make -C test Array_test.ou`.
- For each test, `bin/hcheck` runs `X_test` (or `X_test.script` when present), filters the
  output (masking dates, paths, and `.exe`) into `X_test.ou`,
  and diffs it against `X_test.ref`, leaving `X_test.diff` on a mismatch.
- A `.diff` marks a failing test (a one-line marker if `hcheck` fails without comparing output).
  The `test` target fails while any `.diff` exists, so an unresolved failure is reported on every
  run, and a failing `X_test.ou` is backdated so that the test reruns. All tests still run
  without `make -k`.
- `.ref` files are ground truth. Each test has a single `.ref`, which must match across
  `-O0` through `-O3` and every configuration.
- Floating-point discrepancies from vectorization or sanitizer differences are expected
  and are handled with `round()` wrappers rather than by loosening the comparison.
- Test files use `SHOW()` and `assertx()`, with explicit template instantiations at the
  bottom of the file.

A change is not done until the tests pass and the affected configurations build.

## Validation tiers

Choose the configurations by what a change touches, always with `PEDANTIC=1`:

| Change touches | Validate with |
| -------------- | ------------- |
| Programs, `libHwX`, `libHwWindows`, demo scripts | `unix` and `win`, building only the affected directories; plus the MSBuild build and `make -C demos create check` when rendering or demo outputs may change |
| `libHh` (including the core headers) | also `unix CC=gcc`, and all unit tests under `unix` and `win` |
| `make/`, portability code, or before a checkpoint tag | every configuration, all tests, and the demos |

- `unix` (`.o`) and `win` (`.obj`) keep separate object files, so alternating between them
  stays incremental. `unix`, `mingw`, `cygwin`, and `clang` all share `.o` files (and the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hhoppe/Mesh-processing-library](https://github.com/hhoppe/Mesh-processing-library) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
