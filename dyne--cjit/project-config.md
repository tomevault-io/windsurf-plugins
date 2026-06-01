---
trigger: always_on
description: `cjit` is a small C interpreter and live compiler built around vendored TinyCC in `lib/tinycc`.
---

# AGENTS.md

## Purpose

`cjit` is a small C interpreter and live compiler built around vendored TinyCC in `lib/tinycc`.
It can:

- compile and run C in memory
- compile one source file to an object
- build an executable without running it
- load shared libraries from the host system
- extract embedded runtime assets and tar.gz archives

This file is the maintainer guide for both humans and LLMs. It should be sufficient without reading any planning files.

## Current Reality

Today the codebase is still mostly procedural and centered on `CJITState`.

Main files:

- [src/main.c](/home/jrml/devel/cjit/src/main.c): CLI parsing and top-level dispatch
- [src/cjit.c](/home/jrml/devel/cjit/src/cjit.c): runtime lifecycle, TinyCC setup, source ingestion, linking, execution
- [src/cjit.h](/home/jrml/devel/cjit/src/cjit.h): `CJITState` and public runtime functions
- [src/file.c](/home/jrml/devel/cjit/src/file.c): file/stdin/path helpers
- [src/support/cwalk.c](/home/jrml/devel/cjit/src/support/cwalk.c): low-level path manipulation support used by filesystem and platform adapters
- [src/adapters/cli/ketopt.h](/home/jrml/devel/cjit/src/adapters/cli/ketopt.h): local CLI option parsing dependency
- [src/adapters/platform/build_platform.h](/home/jrml/devel/cjit/src/adapters/platform/build_platform.h): compile-time host and target platform definitions used by core and platform adapters
- [src/adapters/platform/library_resolver_posix.c](/home/jrml/devel/cjit/src/adapters/platform/library_resolver_posix.c): POSIX library resolution and GNU ld script handling
- [src/adapters/platform/library_resolver_windows.c](/home/jrml/devel/cjit/src/adapters/platform/library_resolver_windows.c): Windows DLL resolution
- [lib/muntarfs/muntarfs.h](/home/jrml/devel/cjit/lib/muntarfs/muntarfs.h): bundle extraction surface used by CJIT
- [lib/muntarfs/muntar.c](/home/jrml/devel/cjit/lib/muntarfs/muntar.c): tar extraction and archive reader
- [lib/muntarfs/tinflate.c](/home/jrml/devel/cjit/lib/muntarfs/tinflate.c): low-level deflate implementation
- [lib/muntarfs/tinfgzip.c](/home/jrml/devel/cjit/lib/muntarfs/tinfgzip.c): gzip wrapper over the inflater
- [src/win-compat.c](/home/jrml/devel/cjit/src/win-compat.c): Windows compatibility helpers

Important current behavior:

- `main()` still builds `CJITState` and parses argv, but route request construction now lives in `src/adapters/cli/route_parser.c`.
- `cjit_setup()` is lazy and must happen before compile/link/execute flows.
- non-`SHAREDTCC` builds depend on extracted embedded assets.
- the TinyCC adapter now owns the execute, compile-object, and build-executable flows.
- POSIX execution still forks before running the compiled entrypoint.
- Windows execution still runs in-process.
- stdin execution is supported on POSIX and not supported as the no-file fallback on Windows.
- `-c` currently supports only one source file.
- UTF BOM source files are explicitly rejected.
- source files, requested libraries, library search paths, and resolved library paths now use the `StringList` support wrapper instead of raw `XArray` calls in most runtime code.

## Target Architecture

All new work should move the project toward:

- VSA: slice per use-case
- REPR: request/endpoint/response per CLI route
- Hex: ports/adapters for IO and third-party dependencies
- light DDD: explicit ubiquitous language and invariants

This should stay minimal and C-native. Do not over-engineer it.

### Core use-cases

Use these names consistently:

- `execute-source`
- `compile-object`
- `build-executable`
- `print-status`
- `extract-assets`
- `extract-archive`
- `archive-tool`

### Core vocabulary

- `request`: user intent at the application boundary
- `response`: endpoint result rendered by the CLI
- `session`: mutable runtime/compiler state
- `library request`: logical `-l` input
- `resolved library`: concrete path found by platform logic
- `route`: one CLI action mapped to one request and one response

### Target module layout

Use this direction for refactors:

- `src/app/`: use-case orchestration
- `src/domain/`: requests, responses, errors, core invariants
- `src/adapters/cli/`: argv parsing, dispatch, rendering
- `src/adapters/compiler/`: TinyCC integration
- `src/adapters/fs/`: file/path/tempdir/asset/archive IO
- `src/adapters/platform/`: process and platform-specific library resolution
- `src/support/`: small low-level reusable support only
- `lib/muntarfs/`: standalone bundle pack/extract component and archive implementation for embedded tar/tar.gz assets

Current code is not fully there yet. New changes should avoid making `src/main.c` and `src/cjit.c` even broader.

## Change Map

Use the smallest relevant surface first.

### CLI or argument behavior

Current start point:

- [src/main.c](/home/jrml/devel/cjit/src/main.c)

Target destination:

- `src/adapters/cli/`

### Runtime execute behavior

Current start point:

- [src/cjit.c](/home/jrml/devel/cjit/src/cjit.c)

Target destination:

- `src/app/execute_source.*`

### Compile-to-object behavior

Current start point:

- [src/main.c](/home/jrml/devel/cjit/src/main.c)
- [src/cjit.c](/home/jrml/devel/cjit/src/cjit.c)

Target destination:

- `src/app/compile_object.*`

### Build-executable behavior

Current start point:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dyne/cjit](https://github.com/dyne/cjit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
