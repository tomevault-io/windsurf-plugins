---
trigger: always_on
description: hxcoro is a coroutine library for Haxe that provides generators, async generators, and coroutine primitives. The library supports multiple Haxe compilation targets including JavaScript, C++, HashLink, JVM, Python, PHP, Neko, Eval, and C#.
---

# Copilot Instructions for hxcoro

## Repository Overview

hxcoro is a coroutine library for Haxe that provides generators, async generators, and coroutine primitives. The library supports multiple Haxe compilation targets including JavaScript, C++, HashLink, JVM, Python, PHP, Neko, Eval, and C#.

**Project Type:** Haxe library
**Language:** Haxe
**License:** GPL (see haxelib.json)
**Repository Size:** Small to medium-sized library focused on coroutine implementation

## Build and Test Instructions

### Prerequisites

Nightly Haxe, Neko, and haxelib are **pre-installed** in the agent environment via `.github/workflows/copilot-setup-steps.yml`, so you do not need to install them manually.

All 8 targets are available to run tests.

### Environment Setup

The haxelib setup is handled automatically. You can verify with:

```bash
haxe -version
haxelib list
```

### Running Tests

Tests are located in the `tests/` directory. To run tests for a specific target:

```bash
haxe --cwd tests build-<target>.hxml
```

Where `<target>` is one of: `eval`, `js`, `hl`, `hlc`, `cpp`, `jvm`, `php`, `python`, `neko`, `lua`

**Note:** The CI primarily tests: `eval`, `js`, `hl`, `cpp`, `jvm`, `php`, `python`, `neko`. Other targets like `lua` and `hlc` have build files but may not be regularly tested in CI. The `cs` (C#) target is still in development and should be ignored for now.

**Example:**
```bash
haxe --cwd tests build-eval.hxml  # For eval target
haxe --cwd tests build-js.hxml    # For JavaScript target
```

For debugging mode, add `-debug` flag:
```bash
haxe --cwd tests -debug build-eval.hxml
```

For HXB (Haxe Binary) compilation testing:
```bash
haxe --cwd tests build-<target>.hxml --hxb bin/test.hxb
haxe --cwd tests build-<target>.hxml --hxb-lib bin/test.hxb
```

**C++ Target Specific:** According to the CI workflow, when testing the C++ target, skip the debug mode build (the `-debug` flag variation) and proceed directly with the standard build.

### CI Pipeline

The repository uses GitHub Actions for CI (`.github/workflows/main.yml`). The CI:
- Runs on push and pull requests
- Tests on macOS, Ubuntu, and Windows
- Tests all supported Haxe targets (eval, js, hl, cpp, jvm, php, python, neko)
- Uses concurrency control to cancel in-progress runs when new commits are pushed

**Important:** PHP target is excluded on macOS runners.

## Project Layout and Architecture

### Directory Structure

```
/
├── .github/
│   └── workflows/
│       └── main.yml          # CI configuration
├── src/
│   ├── haxe/                 # Haxe standard library extensions (if any)
│   └── hxcoro/               # Main library code
│       ├── Coro.hx           # Core coroutine utilities and suspend functions
│       ├── CoroRun.hx        # Coroutine execution utilities
│       ├── generators/       # Generator implementations
│       │   ├── Generator.hx         # Synchronous generator (restricted suspension)
│       │   ├── AsyncGenerator.hx    # Asynchronous generator
│       │   ├── YieldingGenerator.hx # Yielding interface
│       │   └── ...          # API-style variants (Es6Generator, CsGenerator, HaxeGenerator)
│       ├── dispatchers/      # Execution dispatchers
│       │   ├── SelfDispatcher.hx
│       │   ├── TrampolineDispatcher.hx
│       │   ├── ThreadPoolDispatcher.hx
│       │   └── LuvDispatcher.cpp.hx  # C++ specific (libuv)
│       ├── schedulers/       # Scheduling strategies
│       ├── continuations/    # Continuation implementations
│       │   ├── CancellingContinuation.hx
│       │   ├── TimeoutContinuation.hx
│       │   └── RacingContinuation.hx
│       ├── concurrent/       # Concurrency primitives
│       │   ├── CoroLatch.hx
│       │   ├── Tls.hx        # Thread-local storage
│       │   └── BackOff.hx    # Used in busy-loops (e.g., CAS loops) to avoid GC deadlocks
│       ├── task/             # Task abstractions
│       ├── exceptions/       # Exception types
│       └── ...
├── tests/
│   ├── build-base.hxml       # Base build configuration
│   ├── build-<target>.hxml   # Target-specific configs (eval, js, cpp, etc.)
│   └── src/                  # Test source files
│       ├── generators/       # Generator tests
│       ├── concurrent/       # Concurrency tests
│       └── ...
├── haxelib.json              # Haxelib package metadata
└── .gitignore
```

### Key Architectural Concepts

1. **Coroutines:** The primary focus of this library. Use the `@:coroutine` metadata on methods. The `Coro` class provides key primitives:
   - `suspend()`: Basic suspension point
   - `suspendCancellable()`: Cancellable suspension with cleanup
   - `delay()`: Async delay primitive

2. **Generators:** A use-case for coroutines that support yielding values. Two main types:
   - `Generator<T, R>`: Synchronous generator requiring Kotlin-style restricted suspension
   - `AsyncGenerator<T, R>`: Asynchronous generator
   - Additional API-style variants (Es6Generator, CsGenerator, HaxeGenerator) provide different interfaces but work on any target

3. **Dispatchers:** Control coroutine execution context. Available dispatchers include:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HaxeFoundation/hxcoro](https://github.com/HaxeFoundation/hxcoro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
