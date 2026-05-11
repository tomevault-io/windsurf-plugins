---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Harding is a class-based Smalltalk dialect that compiles to Nim. It provides a Smalltalk-like object system, Nim compilation backend, REPL, FFI integration, GTK IDE (Bona), and Granite compiler.

**Current Status**: v0.6.0 - Functional interpreter with green threads, MIC/PIC caching, Smalltalk-style resumable exceptions, GTK IDE, VSCode extension, and Granite compiler for native binary compilation.

## Build Commands

```bash
nimble harding           # Build harding REPL in repo root (debug)
nimble harding_release   # Build harding REPL in repo root (release)
nimble bona              # Build bona IDE in repo root (debug)
nimble bona_release      # Build bona IDE in repo root (release)
nimble test              # Run all tests
nimble clean             # Clean build artifacts and binaries
nimble install_harding   # Install harding binary to ~/.local/bin/
nimble install_bona      # Install bona binary and desktop integration
```

**IMPORTANT**: Use `nimble harding` / `nimble bona` to build. These place binaries in the root directory. `nimble build` alone will NOT update root directory binaries.

## Testing

Tests use Nim's unittest framework: `nimble test` runs all tests. Individual tests: `nim c -r tests/test_core.nim`. Test files follow `test_*.nim` pattern covering core interpreter, object model, parser, exceptions, concurrency, stdlib, compiler, and integration. All tests pass with ARC/ORC.

## Logging and Debugging

See `docs/TOOLS_AND_DEBUGGING.md` for full details (VS Code configs, GDB usage, debug build variants).

Both `harding` and `granite` support `--loglevel DEBUG|INFO|WARN|ERROR|FATAL` and `--ast` to dump the AST after parsing.

```bash
harding --loglevel DEBUG myprogram.harding   # Debug logging
harding --ast -e "3 + 4"                     # Show AST
```

Use the `debug` macro from `std/logging` in evaluation code:
```nim
debug("Message send: ", selector)
```

### Debug Builds

```bash
# Build with debug symbols for GDB/LLDB
nim c -d:debug --debugger:native --debuginfo --lineDir:on --stackTrace:on -o:harding_debug src/harding/repl/harding.nim

# Memory debugging
nim c -d:useMalloc -d:debug --debugger:native -o:harding_memdebug src/harding/repl/harding.nim

# Profiling
nim c -d:debug -d:nimprof --debugger:native -o:harding_profile src/harding/repl/harding.nim
```

## Nim Coding Guidelines

### Code Style
- Use camelCase, not snake_case
- Do not shadow the local `result` variable (Nim built-in)
- Doc comments: `##` below proc signature
- Prefer generics or object variants over methods and type inheritance
- Use `return expression` for early exits; prefer direct field access
- **NO `asyncdispatch`** - use threads or taskpools
- Import full modules, not selected symbols
- Use `*` to export public fields
- **ALWAYS** write fmt as `fmt("...")` not `fmt"..."`
- Remove old code during refactoring; keep codebase lean

### Function and Return Style
- **Single-line**: Direct expression without `result =` or `return`
- **Multi-line**: Use `result =` assignment
- **Early exits**: Use `return value`

### Comments
- No comments about how good something is
- No comments reflecting what changed (use git)
- No unnecessary commentary on self-explanatory code

### Memory Management

- **var**: Stack-allocated, copy-on-assignment. Default for most types.
- **ref**: GC-managed heap references. Use for shared objects, `ref object` for types intended to be shared.
- **ptr**: Manual memory (unsafe). Only for FFI. Avoid otherwise.
- Never take address of temporary copies. Never use `addr`/`cast` to create refs from value types in containers.

### ARC/ORC Pointer Safety: Keep-Alive Registries

When storing a Nim `ref` object in `Instance.nimValue` as a raw `pointer`, ARC loses track and may collect it prematurely. Solution: register in a keep-alive seq first.

```nim
registerBlockNode(receiverVal.blockVal)  # Keep alive for ARC
instance.nimValue = cast[pointer](receiverVal.blockVal)  # Now safe
```

Existing registries: `blockNodeRegistry` (types.nim), `processProxies`/`schedulerProxies`/`monitorProxies`/`sharedQueueProxies`/`semaphoreProxies` (scheduler.nim), `globalTableProxies` (vm.nim).

**Rule**: When adding new pointer storage to `nimValue`, always register in the appropriate keep-alive registry first. C/FFI pointers (GTK widgets, file handles) don't need registries.

## Thread Safety

- No asyncdispatch. Use regular threading or taskpools.
- Use `Lock` for concurrent access to shared data structures.
- Use `{.gcsafe.}:` blocks only when code is actually thread-safe (e.g., lock-protected).

### ORC Crash Prevention

Nim ORC can crash with cross-thread circular references (Nim issue #25253). Mitigations:
- Mark types with `{.acyclic.}` for cross-thread ref objects
- Eliminate closures in cross-thread code; use raw pointers instead
- Shutdown controllers BEFORE deinitializing resources they reference

## Stackless VM Design

Harding uses a **stackless VM** driven by a work queue, not the native call stack. This enables green threads and continuations.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gokr/harding](https://github.com/gokr/harding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
