---
trigger: always_on
description: **BME** - a bare-metal x86-64 machine-code viewer / mini step-debugger TUI.
---

# AGENTS.md - bme

## What this is

**BME** - a bare-metal x86-64 machine-code viewer / mini step-debugger TUI.
Paste raw bytes (e.g. `48ffc0`), run them in BME's sandbox, and watch GPRs,
RFLAGS, XMM, and x87 state change one instruction at a time.

- **Goal** - Differential analysis, and the real reason BME exists, is finding
  where x86 *decoders* and the *actual CPU* diverge. A disassembler only maps bytes to a mnemonic. It
  can't know an instruction's runtime effect, and two decoders can disagree on the same bytes -
  executing on real silicon settles it. Some results exist only at runtime (a segment / privilege
  check whose outcome depends on the live descriptor tables and current privilege level can't be
  resolved statically), and some encodings have generation-specific meanings, so identical bytes can
  represent different instructions on different processors.
- **Platform** - Windows and Linux, x86-64 only. CMake hard-errors otherwise.
- **Sandbox model** - Windows reserves guarded mappings and single-steps a sandbox thread with a Vectored Exception Handler. Linux reserves the same mappings in a traced child and uses `PTRACE_SINGLESTEP`. Both capture GPR, RFLAGS, XMM, and x87 state after each completed instruction. Faults, `int3`, and runaway loops are contained and surfaced in the UI rather than crashing the host. Detected Intel SDE or Pin instrumentation prevents execution because native single-step state cannot be trusted.
- The sandbox contains faults and runaway loops, not hostile code. Executed bytes retain user
  privileges and can issue system calls or modify process state.
- Decode is pluggable (`DisasmBackend`) - **Zydis** (default), **bddisasm**, **Capstone**, or **XED**. TUI via FTXUI, CLI via argparse, formatting via fmt.
- x87 80-bit conversions run on the FPU via three small assembly leaves. Windows uses `src/st80.asm` with the Win64 ABI. Linux uses `src/st80.S` with the SysV x86-64 ABI. Both build as the `bme_asm` OBJECT library.

## Layout

```
src/common.hpp       # compile-time compiler, OS, and x86-64 gates
src/util.hpp         # ASCII case-insensitive string comparison
src/bme_core.hpp     # public library API (namespace bme). Types, enums, parse/compose/engine/history/JSON prototypes
src/bme_core.cpp     # platform-neutral library logic, history rendering, CLI, and TUI
src/cpu.hpp          # CPU fingerprint data model and injectable CPUID query contract
src/cpu.cpp          # CPUID/XGETBV collection, decoding, process cache, and summary formatting
src/trace_json.cpp   # isolated Glaze adapter and streaming versioned JSON writer
src/os.hpp           # private VM, environment, clipboard, and stepping contract
src/os.cpp           # shared scratch layout and platform-run preflight
src/os.windows.cpp   # Windows VM, VEH, clipboard, environment, and stepping implementation
src/os.linux.cpp     # Linux mmap, ptrace, terminal clipboard, environment, and stepping implementation
src/main.cpp         # thin entry. Parses args, then calls run_quick / run_tui
src/st80.asm         # x87 80-bit conversion leaves for Win64 MASM
src/st80.S           # x87 80-bit conversion leaves for SysV GNU assembler
test/                # GoogleTest suite for the headless path (links bme_core), built with -DBME_BUILD_TESTS=ON
CMakeLists.txt       # CPM deps, platform targets, version generation, CPack, warning policy
README.md            # user-facing usage, build, package, and license summary
LICENSE              # BME MIT license
THIRD_PARTY_LICENSES.md   # bundled runtime dependency licenses
cmake/GenerateVersion.cmake + bme_version.hpp.in   # --version git tag/hash/url -> generated header
cmake/XED.cmake      # Intel XED backend - CPM download + mfile.py build via ExternalProject
.githooks/pre-commit # rejects unformatted commits (diffs staged content against clang-format, warns if the local major version is not 22)
.github/workflows/   # CI runs clang-format plus Windows and Debian 12 builds
.clang-format        # the authoritative style (clang-format 22)
third_party/cmake/   # CPM.cmake
```

Build dirs matching `cmake-build*` or `build*` are local/gitignored.

## Build

CMake 3.31 or newer is required.

Windows uses Ninja, MASM, and **clang++ targeting MSVC**. **clang-cl** and **MSVC cl** also build through the MSVC ABI path.

```sh
cmake -B build -G Ninja -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++
cmake --build build
```

Linux uses Ninja and a C++23 compiler. Debian 12 with Clang 22 is the CI and packaging baseline. GCC 13 or newer is supported.

```sh
cmake -B build -G Ninja -DCMAKE_C_COMPILER=clang-22 -DCMAKE_CXX_COMPILER=clang++-22
cmake --build build
```

Deps (fmt 12.2.0, argparse 3.2, Glaze 8.3.0, FTXUI 7.0.3, Zydis `a95bb710...`, bddisasm `3.0.1`,
Capstone `5.0.9`) are fetched by CPM. Zydis also builds its pinned Zycore support library. The `URI`
form auto-applies `EXCLUDE_FROM_ALL`/`SYSTEM`, so third-party headers stay out of `-Werror`. Production
Glaze use is private to `bme_serializer`. Tests link it only for generic JSON schema inspection. Intel
XED (`v2026.08.23` plus its mbuild) has no CMake, so CPM downloads it and an ExternalProject builds it
via `mfile.py` (`cmake/XED.cmake`) - needs Python 3.
All slow on first configure.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [angelfor3v3r/bme](https://github.com/angelfor3v3r/bme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
