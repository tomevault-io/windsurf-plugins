---
trigger: always_on
description: cext-review-toolkit is a [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin for reviewing CPython C extensions. It finds API misuse, memory safety bugs, compatibility issues, and correctness problems in code that *consumes* the Python/C API.
---

# CLAUDE.md — cext-review-toolkit development guide

## Project overview
cext-review-toolkit is a [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin for reviewing CPython C extensions. It finds API misuse, memory safety bugs, compatibility issues, and correctness problems in code that *consumes* the Python/C API.

Part of a family of review toolkits:
- [code-review-toolkit](https://github.com/devdanzin/code-review-toolkit) — Python source code
- [cpython-review-toolkit](https://github.com/devdanzin/cpython-review-toolkit) — CPython runtime C code
- **cext-review-toolkit** — C extensions (this project)

Key architectural difference: uses Tree-sitter for C parsing (not regex), enabling analysis that regex cannot do (borrowed-ref lifetime tracking, type slot cross-referencing, accurate scope analysis).

## Prerequisites
- Python 3.10+
- `tree-sitter` and `tree-sitter-c`: `pip install tree-sitter tree-sitter-c`
- `tree-sitter-cpp` (optional): `pip install tree-sitter-cpp` — enables C++ file parsing
- `clang-tidy` and `cppcheck` (optional): system packages — enables external tool cross-referencing
- No other dependencies — all scripts use only the standard library plus tree-sitter

## Dev commands
```bash
# Activate the project venv (Python 3.14 from ~/projects/3.14/python)
source ~/venvs/cext-review-toolkit/bin/activate

# Run all tests
python -m unittest discover tests -v

# Run a specific test file
python -m unittest tests.test_scan_refcounts -v

# Run a single script standalone (all output JSON to stdout)
python plugins/cext-review-toolkit/scripts/scan_refcounts.py /path/to/extension.c
python plugins/cext-review-toolkit/scripts/discover_extension.py /path/to/project

# Lint and format (install ruff/mypy into venv if not present)
ruff format <changed-files>
ruff check <changed-files>
mypy
```

## Code style
- Python 3.10+ (uses `X | Y` union syntax)
- Double quotes for strings
- Type hints on all function signatures
- Docstrings on classes and public functions
- Tests use `unittest` — never pytest
- Linted and formatted with ruff, type checked with mypy

## Project structure

This is a Claude Code plugin, not a pip-installable package.

```
cext-review-toolkit/
├── CLAUDE.md                          # This file
├── README.md                          # User-facing documentation
├── CHANGELOG.md                       # Keep a Changelog format
├── LICENSE                            # MIT
├── cext-review-toolkit-design.md      # Full design document (authoritative reference)
├── .claude/skills/task-workflow/      # Claude Code skill for dev workflow
├── plugins/cext-review-toolkit/       # The actual plugin
│   ├── .claude-plugin/plugin.json     # Plugin metadata
│   ├── agents/                        # 10 agent prompt definitions (markdown)
│   ├── commands/                      # 4 command definitions (markdown)
│   ├── scripts/                       # 11 Python scripts (the core code)
│   └── data/                          # 4 JSON data files (API tables, etc.)
└── tests/                             # unittest test suite
```

## Architecture

### Scripts (the core analysis code)

All scripts live in `plugins/cext-review-toolkit/scripts/`. Every analysis script follows the same pattern: parse C/C++ files with Tree-sitter, find candidate issues, output JSON to stdout.

| Script | Lines | Purpose |
|--------|-------|---------|
| `tree_sitter_utils.py` | ~550 | Core parsing module — all other scripts import from here |
| `scan_common.py` | ~130 | Shared utilities: project root, file discovery, API tables, arg parsing |
| `scan_refcounts.py` | ~360 | Reference counting errors (leaked refs, borrowed-ref-across-call, stolen-ref misuse) |
| `scan_error_paths.py` | ~330 | Error handling bugs (missing NULL checks, exception clobbering, return-without-exception) |
| `scan_null_checks.py` | ~250 | NULL safety (unchecked allocations, deref-before-check) |
| `scan_gil_usage.py` | ~300 | GIL discipline (mismatched macros, API without GIL, blocking with GIL, free-threading) |
| `scan_module_state.py` | ~320 | Module init and state (single-phase init, global state, missing traverse) |
| `scan_type_slots.py` | ~430 | Type definitions (dealloc, traverse, richcompare, flags, heap types) |
| `measure_c_complexity.py` | ~250 | Function complexity scoring |
| `analyze_history.py` | ~520 | Git history analysis (similar bugs, churn prioritization) |
| `discover_extension.py` | ~420 | Extension project layout detection |
| `run_external_tools.py` | ~250 | External tool integration (clang-tidy, cppcheck) |

**Dependency graph:** `tree_sitter_utils.py` is at the center. `scan_common.py` imports from it. All other scripts import from both. `run_external_tools.py` imports only from `scan_common`. No circular dependencies.

**Script calling convention:** Every analysis script exposes `analyze(target: str, *, max_files: int = 0) -> dict` and a `main()` that outputs JSON to stdout. Exception: `analyze_history.py` takes `argv` to match code-review-toolkit conventions.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ReviewToolkits/cext-review-toolkit](https://github.com/ReviewToolkits/cext-review-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
