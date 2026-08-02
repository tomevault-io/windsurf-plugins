---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

V8 is Google's open source JavaScript and WebAssembly engine, written in C++. It implements ECMAScript and is used in Chrome and Node.js. This is a large, performance-critical codebase where correctness bugs often lead to security issues.

## Build System

V8 uses GN (Generate Ninja) as its meta-build system. The primary build wrapper is `tools/dev/gm.py`.

### Common Build Commands

```bash
# Build d8 shell (x64, optimized debug) - recommended for development
tools/dev/gm.py quiet x64.optdebug

# Build d8 shell (x64, debug) - slower, full debug info
tools/dev/gm.py quiet x64.debug

# Build d8 shell (x64, release) - for benchmarking
tools/dev/gm.py quiet x64.release

# Build specific target
tools/dev/gm.py quiet x64.optdebug cctest

# Clean and rebuild
tools/dev/gm.py x64.optdebug.clean
```

**Important**: Always use the `quiet` keyword unless explicitly asked otherwise, to avoid wasting tokens on compilation output. Errors will still be reported.

**Build Modes**:
- `release`: Optimized, no debug info. Use for benchmarking only.
- `debug`: Full debug info, assertions enabled. Slow but essential for debugging.
- `optdebug`: Optimizations + debug info. Best for general development.

**Build Outputs**: Binaries are placed in `out/<arch>.<mode>/` (e.g., `out/x64.optdebug/d8`).

### Alternative: v8gen.py

For custom configurations:
```bash
# Generate build with custom args
tools/dev/v8gen.py x64.release -- v8_enable_slow_dchecks=true
```

## Testing

### Running Tests

The primary test runner is `tools/run-tests.py`:

```bash
# Run all standard tests for a build
tools/run-tests.py --progress dots --exit-after-n-failures=5 --outdir=out/x64.optdebug

# Run specific test suite
tools/run-tests.py --progress dots --exit-after-n-failures=5 --outdir=out/x64.optdebug cctest

# Run specific test file
tools/run-tests.py --progress dots --exit-after-n-failures=5 --outdir=out/x64.optdebug cctest/test-heap

# Run C++ tests only
tools/run-tests.py --progress dots --exit-after-n-failures=5 --outdir=out/x64.optdebug cctest unittests

# Run JavaScript tests
tools/run-tests.py --progress dots --exit-after-n-failures=5 --outdir=out/x64.optdebug mjsunit
```

**Important**: Always use `--progress dots` to minimize output and avoid token waste.

### Test Suites

- **unittests**: Modern C++ unit tests (preferred)
- **cctest**: Older C++ unit tests (being migrated to unittests)
- **mjsunit**: JavaScript language feature tests
- **test262**: Official ECMAScript conformance tests
- **wasm-spec-tests**: WebAssembly specification tests
- **inspector**: DevTools protocol tests
- **message**: Tests for error messages

### Reproducing Test Failures

When a test fails, the output includes a command to reproduce it:
```bash
Command: out/x64.optdebug/d8 --test test/mjsunit/mjsunit.js test/mjsunit/foo.js --random-seed=-190258694
```

You can run this directly or use:
```bash
tools/run-tests.py --progress dots --outdir=out/x64.optdebug mjsunit/foo
```

## Code Structure

### Source Layout (`src/`)

Key directories (see full list in GEMINI.md):
- **`src/api/`**: Public C++ API implementation (headers in `include/`)
- **`src/builtins/`**: JavaScript built-in functions (Array.map, etc.)
- **`src/compiler/`**: TurboFan optimizing compiler + Turboshaft
- **`src/interpreter/`**: Ignition bytecode interpreter
- **`src/baseline/`**: Sparkplug baseline compiler
- **`src/maglev/`**: Maglev mid-tier optimizing compiler
- **`src/heap/`**: Garbage collector and memory management
- **`src/objects/`**: V8 internal and JavaScript object representations
- **`src/codegen/`**: Machine code generation (architecture-specific in subdirs)
- **`src/wasm/`**: WebAssembly implementation
- **`src/execution/`**: Isolates, frames, microtasks, tiering
- **`src/sandbox/`**: V8 sandbox security feature
- **`src/torque/`**: Torque language compiler

### Public API (`include/`)

All public V8 APIs are in `include/`. Main header is `include/v8.h`. These headers are used when embedding V8 (e.g., in Chrome, Node.js).

### Tests (`test/`)

Test suites match their names: `test/cctest/`, `test/unittests/`, `test/mjsunit/`, etc.

## Torque Language

Torque is V8's domain-specific language for writing builtins and object definitions. It compiles to CodeStubAssembler (CSA) code.

### Key Concepts

- **File Extension**: `.tq` (in `src/builtins/` and `src/objects/`)
- **Compilation**: Automatic during build. Generated files go to `out/<build>/gen/torque-generated/`
- **Syntax**: TypeScript-like with V8-specific extensions

### Torque Keywords

- `macro`: Inlined function for reusable logic
- `builtin`: Non-inlined function, callable from JS or other builtins
- `javascript`: Marks a builtin as directly callable from JavaScript
- `transitioning`: Function can cause object map changes
- `extern`: Declares C++ CSA function callable from Torque

### Generated Files

Torque generates multiple file types in `out/<build>/gen/torque-generated/`:
- `*-tq.cc`, `*-tq.inc`, `*-tq-inl.inc`: Implementation files
- `builtin-definitions.h`: List of all builtins
- `class-forward-declarations.h`: Forward declarations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [caoccao/Javet](https://github.com/caoccao/Javet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
