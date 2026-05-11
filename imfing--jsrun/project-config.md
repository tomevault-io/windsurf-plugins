---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

jsrun is a Python library providing JavaScript runtime capabilities via Rust and V8. It exposes a Python API for executing JavaScript code in isolated V8 contexts with async support and permission controls.

**Tech Stack:**
- Rust (core runtime using deno_core)
- Python bindings via PyO3
- Build: Maturin for Python-Rust integration
- Testing: pytest with pytest-asyncio

## Build & Development Commands

The project uses a Makefile for common development tasks. Run `make help` to see all available commands.

### Initial setup
```bash
# Install dependencies and dev tools
make install
```

### Build the project
```bash
# Build development version (debug mode)
make build-dev
```

### Run tests
```bash
# Run all Python tests
make test

# Run tests quietly
make test-quiet

# Run specific test file or pattern
uv run pytest tests/test_runtime.py

# Run tests with asyncio support
uv run pytest tests/test_runtime.py::TestRuntimeAsync -v

# Run Rust tests
cargo test

# Run a single Rust test
cargo test test_runtime_lifecycle
```

### Linting and formatting
```bash
# Auto-format both Rust and Python code
make format

# Lint all code (Rust + Python)
make lint

# Lint Python only
make lint-python

# Auto-fix Python linting issues
make lint-python-fix

# Lint Rust only
make lint-rust
```

### Documentation
```bash
# Build documentation
make docs

# Serve docs locally with live reload
make docs-serve
```

### Standard CI workflow
```bash
# Run the full CI pipeline locally (format, build, lint, test)
make all
```

### Cleanup
```bash
# Remove build artifacts and caches
make clean
```

## Architecture

### Multi-Layer Design

The project has three distinct layers that communicate via well-defined boundaries:

1. **Rust Core** (`src/runtime/`): V8 isolate management, async execution, ops system
2. **Rust-Python Bridge** (`src/runtime/python.rs`, `src/lib.rs`): PyO3 bindings
3. **Python API** (`python/jsrun/__init__.py`): User-facing interface

### Type Conversion Notes

- JavaScript `undefined` now round-trips via the `JsUndefined` sentinel (`jsrun.undefined`), distinct from Python `None` / JS `null`.
- Binary types (`Uint8Array`, `ArrayBuffer`) map to Python `bytes`; Python `bytes`, `bytearray`, and `memoryview` map back to `Uint8Array`.
- Temporal values (`Date` ↔ `datetime`), sets (`Set` ↔ `set`), and arbitrary precision integers (`BigInt` ↔ Python `int`) are handled natively, including op arguments/results.

### Threading Model

Each JavaScript runtime runs on a **dedicated OS thread** with its own:
- V8 isolate (single-threaded, non-Send)
- Tokio single-threaded runtime for async operations
- Command channel for host communication

The main Python thread communicates with runtime threads via message passing (`HostCommand` enum in `runner.rs`).

### Key Components

**RuntimeHandle** (`src/runtime/handle.rs`):
- Clone-safe handle to a runtime thread
- Sends commands via async_mpsc channel
- Does NOT auto-shutdown on drop (explicit `close()` required)
- Thread-safe via Arc\<Mutex\> for shutdown state

**Runtime Thread** (`src/runtime/runner.rs`):
- `RuntimeCoreState` holds the V8 isolate (deno_core JsRuntime) and all runtime data
- `RuntimeDispatcher` processes commands from host thread on the dedicated runtime thread
- Handles promise polling with microtask checkpoints
- Manages JavaScript event loop execution

**Ops System** (`src/runtime/ops.rs`):
- Permission-based host function registry
- Sync and async ops with JSON serialization
- JavaScript calls ops via `__host_op_sync__()` and `__host_op_async__()`

**Module System** (`src/runtime/loader.rs`):
- Static module registration via `add_static_module()`
- Custom module resolution and loading
- Support for both sync and async module evaluation
- ES module imports/exports

**Inspector/Debugger** (`src/runtime/inspector.rs`):
- Chrome DevTools protocol server for debugging
- WebSocket-based inspector sessions
- Runs on dedicated thread with own event loop
- Exposes metadata for connecting devtools frontend

**Snapshot Builder** (`src/runtime/snapshot.rs`):
- Pre-initialize V8 heap state for faster startups
- Execute bootstrap code once at snapshot time
- Create runtime instances from snapshot
- Useful for serverless/multi-tenant scenarios

**Streaming Bridge** (`src/runtime/stream.rs`):
- Bidirectional stream conversion (JS ReadableStream ↔ Python async iterables)
- Non-blocking chunk transfer with backpressure
- Automatic lifecycle management and cleanup
- Stream stats tracking for monitoring

### V8 Platform Initialization

V8 requires exactly one global platform instance. The code uses `OnceCell` to ensure `initialize_platform_once()` is safe to call multiple times. Always call this before creating runtimes (it's done automatically in `Runtime()`).

## Important Implementation Details

### Promise Handling

Async evaluation (`eval_async`) works by:
1. Executing the code and checking if result is a promise
2. If promise: poll via repeated `perform_microtask_checkpoint()` + `yield_now()`
3. Check promise state (Pending/Fulfilled/Rejected)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imfing/jsrun](https://github.com/imfing/jsrun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
