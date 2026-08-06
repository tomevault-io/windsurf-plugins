---
trigger: always_on
description: A Windows-only C11 mod loader for FromSoftware games (`YAFSML`). Elden Ring, Elden Ring Nightreign, and Sekiro are stable; Dark Souls III is experimental. Produces two artifacts:
---

# AGENTS.md — YAFSML

## What this repo is

A Windows-only C11 mod loader for FromSoftware games (`YAFSML`). Elden Ring, Elden Ring Nightreign, and Sekiro are stable; Dark Souls III is experimental. Produces two artifacts:
- `YAFSML.exe` — standalone launcher (`modloader_launcher`, WIN32 subsystem)
- `YAFSML.dll` — injected DLL (`modloader_dll`, proxy for `dxgi.dll` / `dinput8.dll` / `winhttp.dll`)

When comparing or porting behavior from me3, use `docs/me3-repo.md` as the source of truth for the repository, branch, and synced commit. Do not infer the comparison baseline from whichever me3 working tree happens to be checked out. The cache dictionary system intentionally follows the performance optimization in the documented me3 fork; treat it as the project's chosen me3 implementation, not as a parity difference from the original repository.

## Build system

CMake + MSVC (Visual Studio 2026 Community observed in build cache). No Makefile, no npm, no Python build step.

**Configure (out-of-source, debug):**
```
cmake -S . -B cmake-build-debug -G "Visual Studio 18 2026" -DBUILD_TESTING=ON
```

**Build:**
```
cmake --build cmake-build-debug --config Debug
```

**Build release (with LTO + static CRT + stripped binary):**
```
cmake -S . -B cmake-build-release -G "Visual Studio 18 2026" -DCMAKE_BUILD_TYPE=Release
cmake --build cmake-build-release --config Release
```

**Distribution package** (zip + 7z in `<build>/dist/`):
```
cmake --build cmake-build-release --target dist
```

**Run tests (CTest):**
```
ctest --test-dir cmake-build-debug -C Debug --output-on-failure
```

Or build the `RUN_TESTS` target in Visual Studio.

## Build prerequisites

- MSVC (Visual Studio with C/C++ workload)
- NASM — required for `src/modloader/patches/properties_hook.asm`. CMake finds it via `CMAKE_ASM_NASM_COMPILER`. Install via scoop: `scoop install nasm`.

## Source layout

```
src/
  common/         khash_wstr.h — wide-string hash table adapter (shared header, no .c)
  game/           Game registry, descriptors, aliases, and current-process context
  modloader/      Core DLL: config, mod loading, file cache, game hooks, proxy stubs
    patches/      Shared host capabilities plus Elden Ring, Nightreign, Sekiro, and Dark Souls III adapters
    proxy/        Proxy DLL stubs for dxgi/dinput8/winhttp
  launcher/       Standalone EXE: injects the DLL into the game process
  steam/          Steam API helpers (locate game folder via VDF; achievement reset support)
  process/        PE image utilities, signature scanner
deps/
  klib/           khash.h (header-only, INTERFACE target `klib` / alias `klib::headers`)
  minhook/        Function hooking
  inih/           INI parser
  toml-c/         TOML parser (ModEngine2 config compatibility; can be stripped via -DML_STRIP_MODENGINE_CONFIG_SUPPORT=ON)
  getopt/         wingetopt — command-line argument parsing
  lzma/           LZMA SDK (currently commented out in launcher link)
tests/
  smoke_*.c       Smoke and host tests for VFS, hooks, lifecycle, ABI, allocators, config routing, saves, and binary analysis
  test_common.h   Minimal assert macros (EXPECT_TRUE, EXPECT_EQ, EXPECT_STREQ_W, etc.)
```

## Key conventions

**Language:** C11 only. No C++. Headers use `#ifdef __cplusplus extern "C"` guards for potential C++ consumers.

**Memory:** The main loader DLL uses mimalloc. The launcher uses the C runtime allocator. Shared source keeps the existing `LocalAlloc`/`LocalFree` call sites; `src/common/allocator.h`, force-included by each target, maps them to the target allocator. Tests use standard `calloc`/`malloc` via the `kcalloc`/`kmalloc` macros redefined at the top of each test file.

**Hash tables:** `khash.h` (klib) is the only hash table. Two instantiation patterns in use:
- `KHASH_INIT(wstr, ...)` with `kh_wstr_hash_func`/`kh_wstr_hash_equal` — wide-string keys, defined in `src/common/khash_wstr.h`
- `KHASH_MAP_INIT_INT(name, value_type)` — integer keys

Always `#define kcalloc/kmalloc/krealloc/kfree` before `#include "khash.h"` in production files.

**Extension DLLs:** Project-owned extension DLLs have been split out of this repository. External ModEngine extensions continue to use `modengine_ext_init` through the `[dll]` configuration section.

**Signature scanning:** Game function addresses are found at runtime via byte-pattern scanning (`sig_scan`). Patterns use `??` for wildcard bytes. Offsets in comments (e.g. `/* 0x5B8 for version < 1.12 */`) track version-specific differences.

**Version string:** Defined once in `src/CMakeLists.txt` as `YAFSML_VERSION`. Update there only.

**Unicode:** All file paths use `wchar_t`. MSVC `/utf-8` flag is set globally. Non-MSVC targets require `-municode` linker flag (already set in CMakeLists).

## Tests

Tests are lightweight smoke and host tests rather than in-game integration tests. They cover VFS routing and concurrency, Win32 Hook behavior, save mapping, lifecycle and rollback, ABI layouts, allocators, and binary-analysis helpers. No mocking framework is used; failures print to stderr and return non-zero.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soarqin/YAFSML](https://github.com/soarqin/YAFSML) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
