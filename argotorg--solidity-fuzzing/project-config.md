---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build model — two separate build trees

This repo produces two sets of binaries from two different build directories. **They are not interchangeable.**

| Build tree | Toolchain                          | Produces                                                                          |
| ---------- | --------------------------------- | --------------------------------------------------------------------------------- |
| `build/`   | Host compiler, cmake              | `solc`, `sol_debug_runner`, `yul_debug_runner`, `stackshuffler` — for reproducing |
| `build_afl/` | AFL++ `afl-clang-fast++` + system libstdc++ | the proto fuzzers under `tools/ossfuzz/` (+ `sol_afl_diff_runner`) — for fuzzing |

Both trees build natively on the host — **no Docker.** The fuzz build uses AFL++'s `afl-clang-fast++` against the **system libstdc++** (no libc++), so `boost`, `protobuf`+`abseil` come from the system packages and `evmone` is the in-tree static archive built under the AFL toolchain. Only `libprotobuf-mutator` (LPM) is built from source, into `deps_afl/`. AFL++ is a submodule (`AFLplusplus/`) built via the top-level `CMakeLists.txt`; the proto fuzzers link the AFL++ driver (`utils/aflpp_driver/libAFLDriver.a`) as `LIB_FUZZING_ENGINE`.

The proto grammars are mutated by LPM, not AFL's byte-level havoc: `tools/ossfuzz/build_ossfuzz.sh` builds one AFL++ custom-mutator `.so` per grammar (`tools/ossfuzz/lpm_afl_mutator.cc`), and `tools/ossfuzz/run_ossfuzz_afl.sh` wires the matching `.so` into `afl-fuzz` via `AFL_CUSTOM_MUTATOR_LIBRARY` + `AFL_CUSTOM_MUTATOR_ONLY=1`.

### Building fuzzers (build_afl/) — native AFL++

```bash
# Prereq: build the AFL++ toolchain once (afl-clang-fast++ + libAFLDriver.a):
make -C build aflplusplus      # or: make -C AFLplusplus source-only NO_NYX=1

tools/ossfuzz/build_ossfuzz.sh
```

Prerequisites (Arch package names): `clang` + `llvm-dev` (to build AFL++'s LLVM mode), `protobuf` + `abseil` (system, libstdc++), `boost` (static, system), `cmake`, `ninja`, `make`, `git`, `protoc`, `ccache`. The script:

1. builds `libprotobuf-mutator` static + PIC against the **system** protobuf into `deps_afl/` (skipped if `deps_afl/lib/libprotobuf-mutator.a` exists);
2. regenerates `*.pb.{cc,h}` from the `.proto` files with the **system** `protoc` so they match the linked system libprotobuf — these are **git-ignored** (regenerated on every build, not committed);
3. builds one LPM custom-mutator `.so` per grammar (plain `clang++` — the `.so` is loaded by `afl-fuzz` itself, so it must carry no AFL instrumentation);
4. configures `build_afl/` with `afl-clang-fast{,++}` (`-DOSSFUZZ=ON -DLPM_PREFIX=deps_afl -DLIB_FUZZING_ENGINE=…/libAFLDriver.a`) and builds the `ossfuzz_proto` + `ossfuzz_abiv2` targets.

`deps_afl/` and `build_afl/` are git-ignored. To force the LPM rebuild, delete `deps_afl/lib/libprotobuf-mutator.a`.

> **Note:** `tools/ossfuzz/CMakeLists.txt` is AFL-only — configuring `OSSFUZZ` with anything other than `afl-clang-fast++` fails fast with a `FATAL_ERROR`. (The old libc++/libFuzzer build flavour has been removed.)

### Building debug runners and `solc` (build/) — host cmake

```bash
mkdir -p build && cd build
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_EXPORT_COMPILE_COMMANDS=ON \
  -DCMAKE_C_COMPILER_LAUNCHER=ccache -DCMAKE_CXX_COMPILER_LAUNCHER=ccache \
  -DCMAKE_CXX_FLAGS="-fno-omit-frame-pointer" -DCMAKE_C_FLAGS="-fno-omit-frame-pointer" ..
make -j$(nproc)
```

## Architecture

- `solidity/` — git submodule; built as a subdirectory of the top-level `CMakeLists.txt` with `TESTS=OFF`. All fuzzers and runners link against the resulting `solidity`/`libsolc` libraries.
- `evmone/` — git submodule; built as an `ExternalProject`. Runners `dlopen` `libevmone.so` at runtime; its directory is baked into the runner RPATH so `LD_LIBRARY_PATH` is not needed.
- `tools/common/EVMHost.{cpp,h}` — fuzz-specific extensions of solidity's EVMHost (`m_subCallOutOfGas`, `m_contractCreationOrder`). Everything links against this copy, not the one in the solidity submodule.
- `tools/ossfuzz/` — the proto-fuzzer harnesses (run under AFL++ + LPM) and their proto grammars, plus `lpm_afl_mutator.cc` (the LPM→AFL custom-mutator bridge). See `tools/ossfuzz/README.md` for the per-binary breakdown.
- `tools/property/` — fuzztest-based property tests. Two build modes (see top-level `CMakeLists.txt` for the cmake option):
  - **Property mode** (default, `build/` tree, any compiler) — each `FUZZ_TEST` runs as a gtest case with a ~1s random-sampling budget. Useful for CI smoke checks. `--fuzz=...` / `--fuzz_for=...` are no-ops here because the binary lacks coverage instrumentation.
  - **Fuzzing mode** (`build_fuzztest/` tree, clang only) — `cmake -DFUZZTEST_FUZZING_MODE_ENABLED=ON -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ ..` applies `-fsanitize=fuzzer-no-link` + ASan + coverage flags to the whole tree (yul, solidity submodule, and the property target). The resulting binary supports `--fuzz=<Suite>.<Test> [--fuzz_for=<duration>]` for continuous coverage-guided fuzzing with mutator-driven input generation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [argotorg/solidity-fuzzing](https://github.com/argotorg/solidity-fuzzing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
