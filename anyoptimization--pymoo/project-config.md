---
trigger: always_on
description: pymoo is a Python framework for single- and multi-objective optimization (NSGA-II/III, MOEA/D, …): algorithms, test problems, operators, indicators, visualization. **This file is the single source of truth for AI agents and contributors.**
---

# pymoo — Agent Guide

pymoo is a Python framework for single- and multi-objective optimization (NSGA-II/III, MOEA/D, …): algorithms, test problems, operators, indicators, visualization. **This file is the single source of truth for AI agents and contributors.**

## Critical rule — how to run Python

**ALWAYS run Python via `pyclawd python`. NEVER call bare `python` / `python -c`.**

```bash
pyclawd python script.py        # run a script
pyclawd python -m pytest ...     # run a module
pyclawd python -c "import pymoo" # quick check
```

`pyclawd python` runs in the conda env `default` and puts the repo root on `PYTHONPATH`. Bare `python` will miss the env and the in-tree source. (`pyclawd` is the external dev toolkit — `pip install pyclawd`; already installed in the `default` env.)

## Dev commands

`pyclawd` is the single entry point for everything — installed on your PATH in the conda env `default` (`pip install pyclawd`).

| Task | Command |
|---|---|
| Health-check the dev env | `pyclawd doctor` |
| Run a python file/module | `pyclawd python <file>` / `pyclawd python -m <mod>` |
| Fast smoke tier (<30s, xdist) | `pyclawd test fast` |
| Full default gate (skips `long`) | `pyclawd test run` / `pyclawd pytest` |
| Tests by category | `pyclawd test default \| examples \| docs` |
| Run one test / file / dir | `pyclawd test -k "<name>"` · `pyclawd test tests/path::test` · `pyclawd test tests/algorithms/` |
| Lint / autofix | `pyclawd lint` (add `--fix`) |
| Format / check-only | `pyclawd format` (add `--check`) |
| Type-check | `pyclawd typecheck` |
| Aggregate quality gate | `pyclawd check` (format-check → lint → typecheck → test) |
| Build / compile Cython | `pyclawd compile` |
| Clean artifacts | `pyclawd clean` (add `--ext` for compiled) |

`pyclawd check` is the canonical "am I done" gate: it runs format-check → lint → typecheck → test, fail-fast. Tests stop-early with `-x`; rerun only failures with `--lf`. Long tests are excluded by default — pass your own `-m` to override. When in doubt, run `pyclawd doctor` first — it catches the common breakages (missing `jupytext`/`nbconvert`, uncompiled Cython, wrong conda env).

## pyclawd architecture (generic toolkit + project config)

- `pyclawd` is an **external, project-generic** package (`pip install pyclawd`) — it knows nothing about pymoo. ALL pymoo specifics (paths, commands, conda env, deps, test markers/tiers, quality commands, doctor checks) live in **`.pyclawd/config.py`** at the repo root: a module-level `project = Project(...)` (with nested `DocsConfig`, `TestConfig`, `DoctorConfig`, `QualityConfig` — frozen dataclasses from `pyclawd`) plus the `extra_doctor_checks=pymoo_doctor_checks` hook (pymoo version + Cython compiled-extension status).
- **The repo root is the directory containing `.pyclawd/`** — `pyclawd` walks up from cwd to find `.pyclawd/config.py` (override via `--config PATH` or `PYCLAWD_CONFIG`).
- **To change pymoo-specific behavior, edit `.pyclawd/config.py`, NOT the toolkit.**
- **Skills** — the `pyclawd-*` skills (`pyclawd-doctor`, `pyclawd-tests`, `pyclawd-quality`, `pyclawd-docs`) install **globally** at `~/.claude/skills/` and shell out to `pyclawd`; they are available in every project, not vendored here.

## Tests, examples, and docs are all tests

- **Unit tests** live in `tests/`, mirroring the package layout. Markers: `long`, `slow`, `examples`, `docs`, `gradient` (`pytest.ini`, `--strict-markers`).
- **`examples/`** files are executed as integration tests (`pyclawd test examples`). Each example must be self-contained and runnable.
  - **Headless note:** a few examples render a *live animation* via pyrecorder/opencv `cv2.imshow` (`dnsga2.py`, `kgb.py`, `tsp.py`, `stream.py`). On a headless box (no `$DISPLAY`) these need a virtual framebuffer. The `pytest-xvfb` dev dep auto-starts one when `$DISPLAY` is unset — so `pyclawd test examples` just works — **but it requires the system `xvfb` binary** (`apt-get install xvfb`, not a pip dep) and the GUI `opencv-python` (pulled in by `pyrecorder`; do **not** install `opencv-python-headless`, which has no `imshow`). If those two examples fail with a Qt/`xcb` "could not connect to display" or `cv2.imshow ... not implemented` error, that's the missing piece — not a code bug.
- **Docs** are `.md` sources converted to Jupyter notebooks via **jupytext**, executed, then built with Sphinx (`pyclawd test docs`). **Never edit generated `.ipynb` directly** — edit the `.md` source.

## Test pipeline (`pyclawd test`)

Mirrors `pyclawd docs`: a logged, instrumented runner with **fast / comprehensive tiers** and a `--lf` fix-loop. The `pyclawd-tests` skill is the full guide.

| Task | Command |
|---|---|
| Fast smoke tier (<30s, xdist `-n auto`) | `pyclawd test fast` |
| Full unit suite (`run`/`all`/bare `test` are identical) | `pyclawd test run` |
| The fix-list (lastfailed cache) | `pyclawd test failures` |
| Debug the next failure (live, `--lf -x`) | `pyclawd test fix` |
| Slowest tests from last run | `pyclawd test timings [--top N]` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anyoptimization/pymoo](https://github.com/anyoptimization/pymoo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
