---
trigger: always_on
description: labeille hunts CPython JIT bugs by running real-world PyPI package test suites against JIT-enabled builds.
---

# CLAUDE.md — labeille development guide

## Project overview
labeille hunts CPython JIT bugs by running real-world PyPI package test suites against JIT-enabled builds.
Companion to [lafleur](https://github.com/devdanzin/lafleur) (evolutionary JIT fuzzer).

## Security note
labeille installs and runs arbitrary third-party code from PyPI. Never run batch operations on a machine with sensitive credentials or data. Use containers, VMs, or disposable cloud instances for batch runs. See README.md for detailed guidance.

## Environment
- The `.venv` was created with a JIT-enabled CPython build at `~/projects/jit_cpython/python`
- This build has AddressSanitizer (ASAN) enabled — always set `ASAN_OPTIONS=detect_leaks=0`
- The JIT has known bugs that crash dev tools (e.g. mypy) — always set `PYTHON_JIT=0` for dev commands
- Standard prefix for all dev commands: `ASAN_OPTIONS=detect_leaks=0 PYTHON_JIT=0`

## Dev commands
```bash
# Lint and format
ASAN_OPTIONS=detect_leaks=0 PYTHON_JIT=0 .venv/bin/ruff format .
ASAN_OPTIONS=detect_leaks=0 PYTHON_JIT=0 .venv/bin/ruff check .

# Type checking (strict mode, configured in pyproject.toml)
ASAN_OPTIONS=detect_leaks=0 PYTHON_JIT=0 .venv/bin/mypy

# Tests (unittest, NOT pytest)
ASAN_OPTIONS=detect_leaks=0 PYTHON_JIT=0 .venv/bin/python -m unittest discover tests -v
```

## Code style
- Python >=3.13, developed on 3.15
- src layout: `src/labeille/`
- ruff line-length: 99
- mypy strict mode
- Tests use `unittest` with `unittest.mock` — never pytest
- Click CLI framework — Click 8.3+ (no `mix_stderr` in CliRunner)

## Architecture
- `cli.py` — Click CLI entry point with `resolve`, `run`, `scan-deps`, `bisect`, `registry`, `analyze`, `bench`, `ft`, `compat`, `cext-build`, and `tsan-run` subcommands
- `cli_utils.py` — Shared Click helpers (parse_csv_list, parse_env_pairs)
- `resolve.py` — PyPI metadata fetching, repo URL extraction, registry building
- `runner.py` — Clone repos, create venvs, run test suites, detect crashes
- `runner_models.py` — Runner data models (RunnerConfig, PackageResult, RunSummary)
- `repo_ops.py` — Git clone, checkout, sdist install operations
- `crash.py` — Crash detection from exit codes and stderr patterns
- `classifier.py` — Pure Python vs C extension classification from wheel tags
- `bisect.py` — Git bisection to find crash-introducing commits
- `compat.py` — C extension compatibility survey (build, classify failures)
- `compat_cli.py` — Compat CLI subcommands (survey, compare, report)
- `registry.py` — YAML registry I/O (reads from laruche registry or custom path)
- `registry_cli.py` — Batch registry management CLI (add/remove/rename/set fields, validate, migrate)
- `migrations.py` — Registry migration framework (named transformations with logging and dry-run)
- `registry_ops.py` — Batch operations with filtering, atomic writes, dry-run previews
- `analyze.py` — Data loading and analysis (run data, registry stats, comparison, flaky detection)
- `analyze_cli.py` — Analysis CLI (registry, run, compare, history, package subcommands)
- `formatting.py` — Shared text formatting (tables, histograms, sparklines, durations)
- `summary.py` — Run summary formatting (uses formatting.py)
- `scan_deps.py` — AST-based test dependency scanner
- `import_map.py` — Import name to pip package mapping (100+ entries)
- `yaml_lines.py` — Line-level YAML manipulation preserving formatting
- `io_utils.py` — Shared I/O utilities (atomic writes, JSON/YAML loading, process groups)
- `logging.py` — Structured logging setup
- `bench/` — Benchmarking subsystem (runner, config, results, stats, compare, display, export, anomaly, tracking, trends, timing, cache, constraints, system)
- `bench_cli.py` — Benchmark CLI subcommands (run, compare, export, track)
- `ft/` — Free-threading subsystem (runner, results, analysis, compare, display, export, compat)
- `ft_cli.py` — Free-threading CLI subcommands (run, compare, export, compat)
- `tsan_run.py` — TSan-enabled test runner (build venv, install extension, run tests, capture races)
- `tsan_run_cli.py` — TSan-run CLI subcommands (run with --test-script, --quick, --stress)

## Testing notes
- Integration tests mock `fetch_pypi_metadata` to avoid network calls
- Runner tests mock `subprocess.run` and `clone_repo` extensively
- 2224 tests total across 52 test files

## Enriching packages

The package registry lives in [laruche](https://github.com/devdanzin/laruche).
See laruche's README for the enrichment process, field schema, and rules.

Quick setup:
- `labeille registry sync` to fetch the registry
- Default location: `~/.local/share/labeille/registry/`
- `--registry-dir` overrides for any command

Registry batch operations (`labeille registry add-field`, `set-field`, etc.)
work on whichever directory `--registry-dir` points to.

## Performance and resource usage

### Parallel execution
- `--workers N` runs N packages in parallel (default: 1, sequential)
- Each worker spawns its own subprocesses (git, pip, pytest) — threads just wait on child processes, so the GIL is not a bottleneck
- Memory scales linearly with worker count; each test process uses the full interpreter + test suite + any ASAN overhead

### ASAN and resource pressure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devdanzin/labeille](https://github.com/devdanzin/labeille) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
