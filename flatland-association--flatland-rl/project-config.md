---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Flatland is a multi-agent grid/graph rail-network simulator (`RailEnv`) used for Multi-Agent Reinforcement
Learning research, plus tooling around it: rail/line/timetable generation, observations/predictions for RL
policies, rendering, trajectory recording/replay, and evaluation services for the AIcrowd/Flatland challenges.

## Commands

All commands assume dependencies from `requirements-dev.txt` (+ `requirements-ml.txt` for `flatland/ml`/`tests/ml`)
are installed, and are run from the repo root. `tox` wraps most of these into reproducible environments (see
`tox.ini`) — CI (`.github/workflows/checks.yml`)'s containerized jobs run tox's own testenvs directly via
`tox run -e <env> --current-env` (the `tox-current-env` plugin: skips venv creation/dependency reinstall against
an image's already-installed deps, but still applies the testenv's `set_env`/`commands` verbatim), so `tox.ini`
is the single source of truth for both local and CI runs — editing it changes CI behavior without touching
`checks.yml`. Two gotchas this implies: (1) a `[testenv:...]` section that declares its own `set_env` *replaces*
rather than merges with the base `[testenv]` section's `set_env` (tox does not merge these) — any testenv needing
extra env vars must start its `set_env` with `{[testenv]set_env}` or it silently loses `PYTHONPATH`/
`CMAKE_POLICY_VERSION_MINIMUM`; (2) a dev dependency that's only invoked via CLI and never `import`ed (e.g. `tox`,
`tox-current-env`) must be added to the `verify-requirements` testenv's `DEV_MODULES` list or `deptry` (`DEP002`)
flags it as unused.

- **Run the core test suite** (matches CI's `test` job): needs `tests/regression/test_episodes_deadlock_avoidance.py`'s
  and `benchmarks/benchmark_episodes.py`'s fixtures — a `flatland-baselines` checkout on `PYTHONPATH` and a
  `BENCHMARK_EPISODES_FOLDER` populated from the `FLATLAND_BENCHMARK_EPISODES_FOLDER` archive (see
  `flatland-benchmarks-episodes-url` in `checks.yml`):
  ```
  python -m pytest --ignore=tests/ml -m "not slow"
  ```
  Without that fixture set up, drop the benchmark-episode tests or just run a narrower path, e.g.
  `python -m pytest tests/envs/test_foo.py`. If the `flatland-baselines` checkout lives inside this repo's own
  tree (as CI's `test` job does it, and as needed to put it on `PYTHONPATH` without an absolute path), add
  `--ignore=flatland-baselines` — otherwise pytest's default recursive collection picks up its test suite too.
- **Run a single test**: `python -m pytest tests/path/to/test_file.py::test_name`.
- **Run the ML test suite** (`flatland/ml`, RL training — flaky, matches CI's `testml` job): needs `--retries`
  since training runs are inherently non-deterministic:
  ```
  python -m pytest tests/ml --retries 2 --retry-delay 5
  ```
- **Lint**: `flake8 flatland tests examples benchmarks` (config in `tox.ini`'s `[flake8]` section: max line length
  120, `docs` excluded, a fixed ignore list for whitespace/formatting codes). The CI `lint` job is gated on the
  `LINT_ENABLED` repo/org Actions variable (`.github/workflows/checks.yml`'s `if: ${{ vars.LINT_ENABLED ==
  'true' }}`) — unset means disabled — but the config is still the source of truth for style.
- **Regenerate `requirements*.txt`** after changing `pyproject.toml` dependencies: `tox -e requirements`.
- **Check for dependency drift** (unused/missing/misdeclared deps across the `flatland`/`flatland/ml`/`tests`
  boundary): `tox -e py3.13-verify-requirements` (uses `deptry`).
- **Notebooks** (in `notebooks/`, executed as smoke tests): `tox -e py3.12-notebooks`.
- **Full tox matrix**: `tox` (runs everything across Python 3.10–3.14 — slow; prefer targeted `pytest`/`flake8`
  invocations above during iteration).
- **Verify the Cython build actually compiles** (see "Cython-accelerated hot paths" below):
  `tox -e py3.12-verify-cython-build`. To manually build in place and check (Cython auto-provisions via
  `[build-system] requires` - no need to `pip install cython` yourself first): `python -c "from setuptools
  import setup; setup()" build_ext --inplace`, then confirm e.g. `python -c "import
  flatland.envs.step_utils.state_machine as m; assert m.__file__.endswith('.so')"`. Clean up compiled artifacts
  afterward with `rm -rf build && find flatland \( -name '*.c' -o -name '*.so' \) -print0 | xargs -0 rm -f`
  (find-based rather than a hardcoded per-module glob, so it still catches every compiled artifact if an
  `ext-modules` entry moves out of `step_utils` or a new one is added elsewhere under `flatland/`). To force the
  plain-Python fallback instead (e.g. to reproduce
  `verify-build-no-gcc`'s behavior), fake a missing compiler: `CC=/nonexistent-cc CXX=/nonexistent-cxx python -c
  "from setuptools import setup; setup()" build_ext --inplace`.
- **Profiling notebooks** (`benchmarks/flatland_performance_profiling.ipynb`,
  `benchmarks/benchmark_k_shortest_paths_profiling.ipynb`): `tox -e py3.13-profiling` /
  `tox -e py3.13-profiling-get-k-shortest-paths` — use Python 3.13, not 3.12 (see the Cython section below for

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flatland-association/flatland-rl](https://github.com/flatland-association/flatland-rl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
