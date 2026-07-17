---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

`libhaze` is a CUDA-shaped record-and-replay runtime for the Niobium FHE
accelerator. The public C ABI (`include/haze/haze.h`) mirrors CUDA function
shapes one-for-one (`cudaMalloc` → `hazeMalloc`, `cudaMemcpy` → `hazeMemcpy`,
`cudaStreamSynchronize` → `hazeStreamSynchronize`, etc.) so codebases written
against CUDA — primarily FIDESlib — can port by mechanical prefix substitution.

The implementation does not execute polynomial math. Every compute call records
a node into FHETCH IR via `niobium::fhetch::sr_*`. Outputs are explicit: declare
each result with `hazeTagOutput`, then `hazeFlush` finalizes the trace, dispatches it to a backend
(in-process simulator or HTTP transport to `nbcc_fhetch_replay`), and writes the
simulator-computed values into the tagged outputs' shadow buffers. `hazeMemcpy(D2H)`
is then a pure shadow read; reading an address that was not tagged-and-flushed
returns `HAZE_ERROR_NOT_FLUSHED`.

## Toolchain

Required to build, test, and lint:

- A C++23 compiler. Clang (the version nixpkgs-unstable currently ships) is
  what CI tests against; `-Wthread-safety` (the canonical enforcement path for
  the lock contracts in `src/common/thread_safety.hpp`) only fires under clang.
  Clang 19 is the supported floor.
- CMake >= 3.22.
- Catch2 v3 (`Catch2::Catch2WithMain`), discovered via `find_package`.
- For lint/format: clang-format, clang-tidy, clangd (any version that
  understands C++23). Configs in `.clang-format`, `.clang-tidy`, `.clangd`.

If a tool is missing, prefer acquiring it via the project's nix flake
(below) or via `nix-shell -p <pkg>` / `nix shell nixpkgs#<pkg>`. Avoid
`brew install` / `apt install` of build tools unless that is the only
option for the host; the flake is the source of truth for the versions
the project tests against.

### Preferred path: nix flake

If `nix` is installed, use it. The flake provides a hermetic devshell
with everything pinned (clang via `clangStdenv`, cmake, clang-tools,
catch2_3, jujutsu, nixfmt, clang-tidy-cache):

```sh
cd /path/to/niobium-haze
nix develop                                    # interactive shell
nix develop --command bash -c '<command>'      # one-shot
```

To add a new tool the flake doesn't yet provide, edit
`devShells.default.nativeBuildInputs` in `flake.nix` rather than
installing it globally.

### Bare path: build without nix

The Makefile makes no assumption that nix is present. Install the
toolchain via the host package manager and run `make` directly.

macOS (Homebrew):

```sh
brew install llvm cmake catch2
# Only if the host's default clang is < 19:
brew link --force --overwrite llvm
export CC=$(brew --prefix llvm)/bin/clang CXX=$(brew --prefix llvm)/bin/clang++
```

Linux (Debian/Ubuntu) — clang-19 is the minimum the project supports; pick a
newer apt package (`clang-20`, `clang-21`, ...) where available to match what
nix-based CI tests with:

```sh
sudo apt install clang-19 cmake catch2 clang-format clang-tidy clangd
```

Then build/test/lint per the next section.

### macOS SDK / ABI mismatch trap

This trap triggers only when **mixing** nix and non-nix builds — for
example, OpenFHE / `libnbfhetch` built from a host shell and haze built
inside `nix develop` (or vice versa). A pure-nix or pure-host workflow
is unaffected.

Symptom: a clean release build links with warnings like `object file ...
was built for newer macOS version (26.0) than being linked (14.0)`, then
segfaults non-deterministically inside calls that should be no-ops
(`hazeConfigureDevice`, `hazeMalloc`). Two libc++ ABI versions are
colliding in the same process.

Fix: rebuild every dylib in the link graph in the same shell so each
carries the same `LC_BUILD_VERSION minos`. From inside `nix develop`:

```sh
rm -rf build dbuild
EXTERNAL_OPENFHE=1 make build
```

If the warnings persist, wipe and rebuild OpenFHE itself in the same
shell. Verify with `otool -l <dylib> | grep -A3 LC_BUILD_VERSION` —
`minos` must agree across `libhaze.dylib`, `libnbfhetch.dylib`, and every
`libOPENFHE*.dylib`. Any non-trivial test segfault accompanied by these
linker warnings is this trap until proven otherwise; do not bisect
before checking.

## Build, test, lint

Top-level `Makefile` is the standalone entry point. `MODE=debug`
(default) selects `dbuild/`; `MODE=release` selects `build/`. CI
forces `MODE=release` (via `build-test.yml`'s explicit flag and via
the flake derivations, which build `haze` with
`-DCMAKE_BUILD_TYPE=Release`) so the gates exercise the optimization
level that ships; local iteration defaults to debug so editor
diagnostics, asserts, and sanitizer turnaround line up with the
binary that gets produced.

```sh
make sync                      # init vendor/niobium-fhetch (recursive)
make build                     # configure + build (debug; build/ for release)
make test                      # test-unit + test-sim (default)
make test-unit                 # ~[integration] tag, HAZE_TARGET=local
make test-sim                  # [integration] tag, in-process FHETCH simulator
make test-transport NIOBIUM_COMPILER_ROOT=/path/to/niobium-compiler

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NiobiumInc/niobium-haze](https://github.com/NiobiumInc/niobium-haze) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
