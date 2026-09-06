---
trigger: always_on
description: > Cognitive complexity analyzer for Python - measures how hard code is for humans to understand.
---

# complexipy

> Cognitive complexity analyzer for Python - measures how hard code is for humans to understand.

This is the single source of truth for agent instructions in this repository.
`CLAUDE.md` is a pointer to this file plus a short Claude-Code-specific section -
put anything tool-agnostic here, not there. See [Keeping This File Current](#keeping-this-file-current).

## Tech Stack

- **Language:** Python 3.8+ (package shell) + Rust (engine, CLI, diff)
- **Framework:** clap (CLI args), owo-colors/syntect/comfy-table (terminal output)
- **Package Manager:** uv (Python), Cargo (Rust)
- **Build:** maturin (Rust → Python extension), wasm-pack (Rust → WASM)
- **Docs:** MkDocs Material (EN + ES)

The analysis engine is Rust; the CLI and public Python API are thin wrappers over a
PyO3 extension module (`complexipy._complexipy`). The same Rust core also compiles to
WASM for the browser demo and the VS Code extension. Scoring follows G. Ann Campbell's
SonarSource cognitive complexity paper.

## Project Structure

```
complexipy/
├── crates/                       # Cargo workspace (root Cargo.toml is virtual)
│   ├── complexipy-core/          # engine: algorithm, types, rules, runner, diff
│   │   └── src/
│   │       ├── cognitive_complexity.rs   # AST walking + scoring algorithm
│   │       ├── classes.rs                # Data types (FunctionComplexity, RefactorPlan, ...)
│   │       ├── refactor_plans.rs         # ComplexityRegion tree + build_refactor_plans()
│   │       ├── rules/                    # Clippy-style refactor rule system
│   │       │   ├── types.rs              # RefactorRule trait + RuleMetadata
│   │       │   ├── complexity.rs         # Concrete rules (C001-C007, C011)
│   │       │   └── registry.rs           # Registration, filtering, ranking, overlap
│   │       ├── runner.rs                 # File/dir/git-URL walk + shared entry points
│   │       ├── diff.rs                   # git-diff comparison (compute_diff, DiffEntry)
│   │       ├── api.rs                    # Rust-level code_complexity / file_complexity
│   │       ├── utils.rs                  # CSV/JSON writers, snapshot I/O, AST helpers
│   │       └── helpers/exclude.rs        # Glob-based file exclusion
│   ├── complexipy-cli/           # CLI: clap args, output rendering, run orchestration
│   ├── complexipy-python/        # PyO3 module (_complexipy) + py_diff wrappers
│   └── complexipy-wasm/          # wasm-bindgen entry point
│
├── complexipy/                   # Python package: thin re-export layer over Rust
│   ├── __init__.py               # Public API: imports _complexipy, file_complexity wrapper
│   ├── cli.py                    # Console-script bootstrap → _complexipy.run_cli
│   ├── py.typed                  # PEP 561 marker
│   └── _complexipy.pyi           # Type stubs for the Rust extension
│
├── tests/                        # pytest test suite
│   ├── main.py                   # Core tests + paper conformance
│   ├── src/                      # Test fixture .py files (excluded from collection)
│   ├── fixtures/refactor_plans/  # Rule-behaviour fixtures
│   └── test_*.py                 # Utility module tests
│
├── docs/                         # MkDocs content (EN + es/)
├── web/                          # Browser demo (WASM + CodeMirror)
├── vscode/                       # VS Code extension (WASM module)
└── .github/workflows/            # CI, PR title check, release
```

## Commands

### Setup

```bash
uv sync
```

### Build (Rust extension)

```bash
uv run maturin develop
```

**After editing any `crates/**/*.rs`, rebuild before running pytest** - otherwise
pytest exercises the previously built `.so`, and both passing and failing results
are meaningless.

### Test

```bash
uv run pytest               # Python suite (testpaths = tests/)
cargo test --workspace      # Rust tests across all four crates
```

Single test:

```bash
uv run pytest tests/main.py::TestFiles::test_match
uv run pytest tests/test_refactor_plans.py::test_match_dispatcher_creates_dispatcher_plan
uv run pytest -k refactor
cargo test -p complexipy-core rules::registry
```

### Lint, Format & Type Check

```bash
uv run ruff check .
uv run ruff format .
uv run ty check .
```

### Cross-target compile checks

CI builds wheels for both features but only type-checks and tests the Python one;
break the wasm target and *release* breaks, not CI.

```bash
cargo check -p complexipy-core
cargo check -p complexipy-core --no-default-features
cargo check -p complexipy-wasm --target wasm32-unknown-unknown
```

### Run

```bash
uv run complexipy <path>
uv run complexipy . --diff main --max-complexity-allowed 15
uv run complexipy complexipy --failed          # dogfood the tool on itself
```

### Benchmarks

```bash
./benchmarks/benchmark-cli.sh # corpus comparison plus synthetic scaling guard
```

`benchmark-cli.sh` compares the current CLI against the 7.0.1 baseline on
pinned real repos, then times a generated synthetic fixture at 1x/2x/4x
sizes (generated into `~/.cache/complexipy-benchmarks/scaling/`, never
committed) and records the scaling ratios in `benchmarks/results.md`,
which the docs pages include via pymdownx snippets.

### WASM / web demo

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rohaquinlop/complexipy](https://github.com/rohaquinlop/complexipy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
