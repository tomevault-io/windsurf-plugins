---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project

A C++20 reimplementation of [quilt](https://savannah.nongnu.org/projects/quilt), the patch management tool.
Builds a single `quilt` binary that manages a stack of patches against a source tree. Public domain (Unlicense).

The reference document for quilt behavior is `docs/manual.md`. When in doubt about how a command should behave, run real `quilt` (system-installed) through the same scenario and match its output.

## Dependencies

Install the original quilt for behavioral comparison and test validation:

```bash
sudo apt-get install -y quilt
```

The test suite can be run against real quilt (`bash test/test.sh quilt`) to verify test correctness.

For Windows cross-compilation and testing, install mingw-w64 and wine:

```bash
sudo apt-get install -y mingw-w64 wine
```

**apt install tips**: Always use `-y` to skip interactive confirmation prompts. If package lists are stale, run `sudo apt-get update` first. These installs (especially `wine`) can be slow — run them as background tasks when possible.

## Build

```bash
# Linux (native)
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build

# Windows cross-compile (mingw-w64, produces static .exe)
cmake -B build-win32 -DCMAKE_TOOLCHAIN_FILE=cmake/mingw-w64.cmake
cmake --build build-win32
```

## Amalgamation

The build also produces `quilt.cpp` (project root) — a self-contained single-file
amalgamation of all sources using `platform_win32.cpp`. Compile it standalone with:

```bash
# Cross-compile to Windows (mingw-w64)
x86_64-w64-mingw32-g++ -std=c++20 -o quilt.exe quilt.cpp -static -lshell32

# Native Windows g++
g++ -std=c++20 -o quilt.exe quilt.cpp -lshell32
```

Regenerate: `cmake --build build --target amalgam` (not built by default).
Do not edit `quilt.cpp` directly.

## Test

```bash
cmake --build build
ctest --test-dir build -j8
```

CTest now registers one test per scenario, so failures are isolated and the suite can run in parallel. The CTest path is shell-free and uses CMake scripting under `test/`.

To run the suite against an arbitrary quilt binary, configure with `QUILT_TEST_EXECUTABLE`:

```bash
cmake -B build-external -DQUILT_TEST_EXECUTABLE=/path/to/quilt
ctest --test-dir build-external -j8
```

The legacy shell harness remains in `test/test.sh` for ad hoc comparison, including against real quilt (`bash test/test.sh quilt`).

## Architecture

All internal strings are UTF-8. Indices and counts use `ptrdiff_t` (signed) with `std::ssize()` instead of `.size()`. The boundary utility `checked_cast<T>()` is used at every signed-to-unsigned conversion point, along with `str_find()` and `str_rfind()` (→ `ptrdiff_t`, returning −1 for not-found), all defined in `quilt.hpp`. `size_t` is only used at system call boundaries (POSIX `write`, Win32 `WriteFile`).

### Headers

- `quilt.hpp` — precompiled header and shared interface. Defines `QuiltState`, string/path utilities, `Command` dispatch table type, and all `cmd_*` function declarations.
- `platform.hpp` — platform abstraction layer: process execution (`run_cmd`, `run_cmd_input`), filesystem ops, environment, and I/O. Declares `quilt_main()`.

### Source files

- `core.cpp` — `QuiltState` methods, string/path utilities, series/applied-patches file I/O, backup/restore file helpers, `quilt_main()` entry point with command dispatch table.
- `cmd_stack.cpp` — stack navigation and push/pop: series, applied, unapplied, top, next, previous, push, pop.
- `cmd_patch.cpp` — patch content commands: new, add, remove, edit, refresh, diff, revert, snapshot, init.
- `cmd_manage.cpp` — patch management: delete, rename, import, header, files, patches, fold, fork, upgrade.
- `cmd_mail.cpp` — mbox generation for emailing patches (`quilt mail`).
- `cmd_annotate.cpp` — annotated file listing showing which patches modify which lines.
- `cmd_graph.cpp` — dependency graph generation in dot(1) format.
- `patch.cpp` — built-in patch engine for applying unified diffs (fuzz, reverse, merge conflicts, reject files).
- `cmd_stubs.cpp` — unimplemented commands that return "not yet implemented": grep, setup, shell.
- `platform_posix.cpp` — POSIX implementation (fork/exec, POSIX file I/O). Contains `main()`.
- `platform_win32.cpp` — Win32 implementation (`CreateProcess`, wide-char APIs, UTF-16 conversion). Contains `main()`.

### Key design patterns

- **Platform selection at build time**: `CMakeLists.txt` links exactly one of `platform_posix.cpp` or `platform_win32.cpp`. No `#ifdef` in shared code.
- **Backup-based patch tracking**: Push/pop works by backing up files into `.pc/<patchname>/` before applying patches. Pop restores from these backups. A built-in patch engine (`patch.cpp`) applies unified diffs; the external `diff` command is used for generating them.
- **Metadata files in `.pc/<patch>/`**: The `.timestamp` and `.needs_refresh` files are quilt metadata, not tracked files. `files_in_patch()` filters these out (anything starting with `.`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skeeto/quilt.cpp](https://github.com/skeeto/quilt.cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
