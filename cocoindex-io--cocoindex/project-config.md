---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/claude-code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/claude-code) when working with code in this repository.

## Build and Test Commands

This project uses [uv](https://docs.astral.sh/uv/) for Python project management.

### Building

```bash
uv run maturin develop   # Build Rust code and install Python package (required after Rust changes)
```

### Testing

```bash
cargo test               # Run Rust tests
uv run mypy              # Type check Python code
uv run pytest python/    # Run Python tests (use after both Rust and Python changes)
```

### Code Formatting and Linting

```bash
uv run ruff format .           # Format Python code
uv run ruff format --check .   # Check formatting without making changes (same as CI)
uv run ruff check .            # Lint Python code
```

### Workflow Summary

| Change Type | Commands to Run |
|-------------|-----------------|
| Rust code only | `uv run maturin develop && cargo test` |
| Python code only | `uv run mypy && uv run pytest python/` |
| Both Rust and Python | Run all commands from both categories above |
| Python formatting | `uv run ruff format .` |

## Code Structure

```
cocoindex/
├── rust/                       # Rust crates (workspace)
│   ├── core/                   # Core engine crate
│   │   └── src/
│   │       ├── engine/         # Core engine
│   │       ├── state/          # States of the core engine
│   │       └── inspect/        # Database inspection utilities
│   ├── py/                     # Python bindings (PyO3)
│   ├── py_utils/               # Python-Rust utility helpers (error, convert, future)
│   ├── utils/                  # General utilities: error, batching, fingerprint, etc.
│   └── ops_text/               # Text processing operations (splitter, language detection)
│
├── python/
│   ├── cocoindex/              # Python package
│   │   ├── __init__.py         # Package entry point
│   │   ├── cli.py              # CLI commands
│   │   ├── _internal/          # Internal implementation for the core engine
│   │   │   ├── api.py          # Public API: mount, use_mount, mount_each, map, mount_target, App, fn, start/stop
│   │   │   ├── app.py          # App base implementation
│   │   │   ├── context_keys.py # ContextKey and ContextProvider
│   │   │   ├── environment.py  # Environment and lifespan handling
│   │   │   ├── function.py     # @coco.fn decorator implementation
│   │   │   ├── component_ctx.py # ComponentContext and component_subpath
│   │   │   ├── target_state.py # Target state implementation
│   │   │   └── core.pyi        # Type stubs for the Rust extension module (update when PyO3 APIs change)
│   │   ├── connectors/         # External system connectors (localfs, postgres, qdrant, lancedb, google_drive)
│   │   ├── connectorkits/      # Connector building utilities
│   │   ├── resources/          # Abstractions: file.py (FileLike), chunk.py (Chunk), schema.py
│   │   └── ops/                # Operations: text.py (RecursiveSplitter), sentence_transformers.py
│   └── tests/                  # Python tests
│
├── examples/                   # Example applications
├── docs/                       # Documentation
└── dev/                        # Development utilities
```

## Key Concepts

### Mental model: declarative data pipelines

CocoIndex uses a **declarative** programming model — you specify *what* your output should look like (target states), not *how* to incrementally update it. The engine handles change detection and applies minimal updates automatically.

Think of it like:

* **React**: declare UI as function of state → React re-renders what changed
* **Spreadsheets**: declare formulas → cells recompute when inputs change
* **CocoIndex**: declare target states as function of source → engine syncs what changed

### Core concepts

**App** — The top-level runnable unit. Bundles a main function with its arguments. When you call `app.update()`, the main function runs as the root processing component.

**Processing Component** — The unit of execution that owns a set of target states. Created by `mount()` or `use_mount()` at a specific component path. When a component finishes, its target states sync atomically to external systems.

**Component Path** — Stable identifier for a processing component across runs. Created via `coco.component_subpath("process", filename)`. CocoIndex uses component paths to:

* Match components to their previous runs for change detection
* Determine ownership of target states (if a path disappears, its target states are cleaned up)

**Target State** — What you want to exist in an external system (a file, a database row, a table). You *declare* target states; CocoIndex keeps them in sync — creating, updating, or removing as needed.

**Target** — The API object used to declare target states (e.g., `DirTarget`, `TableTarget`). Targets can be nested: a container target state (directory/table) provides a Target for declaring child target states (files/rows).

**Function** — A Python function decorated with `@coco.fn`. Use `memo=True` to enable memoization (skip execution when inputs and code are unchanged).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cocoindex-io/cocoindex](https://github.com/cocoindex-io/cocoindex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
