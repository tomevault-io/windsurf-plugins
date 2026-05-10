---
trigger: always_on
description: Instructions for AI coding agents (Codex, Devin, Copilot Workspace, etc.) working in this repository.
---

# AGENTS.md — WhiskeySour

Instructions for AI coding agents (Codex, Devin, Copilot Workspace, etc.) working in this repository.

---

## What this project is

WhiskeySour is a Rust-backed, drop-in replacement for Python's BeautifulSoup library. It exposes an identical public API to BS4 but parses HTML 7–11× faster, queries 7–14× faster, serialises 43–50× faster, and uses 12× less memory per node.

The project is in **Beta**. The core implementation is complete and 450 unit tests pass.

---

## Build system

This is a **Rust + Python hybrid project** built with [maturin](https://www.maturin.rs). You cannot run the Python tests without first compiling the Rust extension.

### Prerequisites
- Python 3.9+, virtual environment at `.venv/`
- Rust toolchain installed at `~/.cargo/bin/`

### Build the extension

```bash
source .venv/bin/activate
PATH="$HOME/.cargo/bin:$PATH" maturin develop          # dev build (default for tests)
PATH="$HOME/.cargo/bin:$PATH" maturin develop --release # release build (required for benchmarks)
```

### Type-check Rust without building

```bash
~/.cargo/bin/cargo check -p whiskeysour-py
```

### Lint and format

```bash
~/.cargo/bin/cargo fmt                        # format Rust
~/.cargo/bin/cargo clippy -- -D warnings      # lint Rust; warnings are errors
```

---

## Running tests

Always activate the venv first. Pass `--override-ini="addopts="` to suppress benchmark-only defaults.

```bash
# Unit tests (primary suite — must all pass before any PR)
source .venv/bin/activate
.venv/bin/pytest tests/python/unit/ --override-ini="addopts=" -q

# Skip slow tests (large documents / deep nesting)
.venv/bin/pytest tests/python/unit/ --override-ini="addopts=" -q -m "not slow"

# Integration tests (BS4 API parity — requires beautifulsoup4)
.venv/bin/pytest tests/python/integration/ --override-ini="addopts=" -q

# Fuzz / property-based tests (requires hypothesis)
.venv/bin/pytest tests/python/fuzz/ --override-ini="addopts=" -v

# Check compatibility against real BS4 (no file changes needed)
PYTHONPATH=/tmp .venv/bin/pytest tests/python/unit/ tests/python/integration/ \
  --override-ini="addopts=" -q --tb=no
```

### Test pass-rate contract

| Suite | Minimum pass rate |
|-------|:-----------------:|
| `unit/` | **450 / 451** (1 skipped is expected) |
| `integration/` | **58 / 58** |

Do not submit changes that reduce these numbers.

---

## Project structure

```
WhiskeySour/
├── Cargo.toml                      # Rust workspace (two crates)
├── pyproject.toml                  # maturin config; Python package = whiskeysour
├── pytest.ini
│
├── crates/
│   ├── whiskeysour-core/            # Pure Rust library — no Python dependency
│   │   └── src/
│   │       ├── node.rs             # NodeId (u32), NodeData enum, Attr, arena types
│   │       ├── document.rs         # Document: flat Vec<Node> arena
│   │       ├── parser/             # html5ever TreeSink integration
│   │       ├── selector/           # CSS DFA + LRU cache
│   │       ├── traversal/          # Iterator types for tree walks
│   │       ├── query/              # find / find_all / select
│   │       └── serialize/          # HTML output + prettify
│   │
│   └── whiskeysour-py/
│       └── src/lib.rs              # PyO3 bindings: _Tag, _Document Python classes
│
├── python/whiskeysour/
│   ├── __init__.py                 # Full BS4-compatible Python API
│   └── _core.pyi                  # Type stubs
│
└── tests/python/
    ├── conftest.py                 # Shared fixtures (parse, parse_fragment, html_doc)
    ├── unit/                       # 450 tests across 10 files
    ├── integration/                # 58 BS4 compatibility tests
    ├── performance/                # Benchmarks + bench_comparison.py
    └── fuzz/                       # Hypothesis property tests
```

---

## Architecture: three layers

These layers must remain separate. Do not merge their responsibilities.

### Layer 1 — `whiskeysour-core` (pure Rust)
The tree is a flat `Vec<Node>` arena. A `NodeId` is a `u32` index — no pointers, no `Rc`, no `Box`. Each `Node` carries a `NodeData` enum variant: `Document`, `Element`, `Text`, `Comment`, `CData`, `ProcessingInstruction`, or `Doctype`.

Element attributes use `SmallVec<[Attr; 4]>` — zero heap allocation for elements with ≤4 attrs (which covers the overwhelming majority of real HTML).

This crate has no PyO3 dependency and must stay that way. It can be used as a standalone Rust library.

### Layer 2 — `whiskeysour-py/src/lib.rs`
PyO3 glue code only. `PyTag` = `Arc<RwLock<Document>>` + `NodeId`. Cloning is cheap. All non-trivial Rust work releases the Python GIL with `py.allow_threads(...)`.

### Layer 3 — `python/whiskeysour/__init__.py`
The BeautifulSoup-compatible Python shim. Key components:
- `Tag` — wraps `_Tag` (a `PyTag`), exposes the full BS4 API.
- `NavigableString` — a `str` subclass with `name = None` (class attribute) and navigation properties.
- `_AttrProxy` — a `dict` subclass that syncs mutations back to Rust. **Lazy**: only created when `.attrs` is accessed.
- `_wrap(rust_obj)` — dispatches `node_type` to `Tag`, `NavigableString`, `Comment`, etc.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [the-pro/WhiskeySour](https://github.com/the-pro/WhiskeySour) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
