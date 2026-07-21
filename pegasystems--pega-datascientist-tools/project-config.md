---
trigger: always_on
description: handles single files, directories (Hive-partitioned layouts),
---

# Agent Guide for pega-datascientist-tools

This repo is a Python library and tooling suite for Pega data science.
Primary code lives under `python/pdstools`, with tests in `python/tests`.

Keep edits focused, run the narrowest tests you can, and prefer `uv` for
dependencies and execution.

## Critical rules

- **Open source repo.** Never include customer names, customer data, or
  internal project names anywhere (code, comments, commit messages, tests).
- **Never push without repo-local pre-commit installed and run.** Before
  pushing any branch, install hooks in that checkout with
  `uv run pre-commit install` and run `uv run pre-commit run --all-files`
  using this repo's `.pre-commit-config.yaml`. If `pre-commit` is not
  installed yet, stop and prompt the user to allow installing the dev
  tooling (`uv sync --extra dev`) rather than pushing unhooked.
- **Git workflow.** Do not `git commit` on the user's working branch.
  Stage with `git add` and let the user commit. Exception: branches you
  created yourself, or when the user explicitly asks for a commit/PR.
- **Stay focused, but fix tightly-coupled bugs.** Don't fix
  pre-existing issues unrelated to the task at hand. However, if a
  refactor surfaces a bug that is directly caused by or tightly
  coupled to the code you're changing (e.g. a wrong return-type
  annotation that callers were defensively working around, a stale
  `# type: ignore` that hid a real mismatch), fix it in the same PR
  and call it out in the commit message. Splitting it off creates a
  regression window where the fix lacks the surrounding context.
- **Product naming.** User-facing text: "Decision Analysis Tool",
  "ADM Health Check". Code/CLI: `decision_analyzer`, `adm_healthcheck`.
- **`verbose` parameters.** Reserve `verbose` for genuine user-facing
  progress — tqdm progress bars on long-running I/O, multi-stage CLI
  output (`"Step 1/3: writing parquet…"`). For anything that prints
  internal technical detail (subprocess output, decoded values, "file
  not found in dir X", schema warnings), drop the parameter and use
  `logger.debug()` / `logger.info()` instead. Never overload `verbose`
  to control non-output behaviour (e.g. cache invalidation, retry
  policy) — that's a separate parameter.
- **`show` parameters on plot methods are an anti-pattern.** Plot
  methods must only build and return the figure; the caller decides how
  to display it. Adding `show: bool = True` that calls `fig.show()`
  internally causes Jupyter to render the figure twice (once from
  `fig.show()`, once from the cell's implicit display of the return
  value). Never add `show` to a plot method. For non-Plotly objects
  that Jupyter cannot auto-render (e.g. `pydot.Graph`), unconditionally
  call `display()` when IPython is available, without gating on a
  `show` parameter.
- **Plotly label/title clipping checklist.** Always do a visual check
  on exported HTML; Plotly's default margins are tight and common
  causes of clipping are:
  - Long y-axis tick labels on horizontal bar charts → add
    `yaxis_automargin=True` to `update_layout()`.
  - Left y-axis title (rotated text) cut off → `margin=dict(l=90)`.
  - Right secondary y-axis title cut off → `margin=dict(r=120)`.
  - `updatemenus` button bar overlapping the chart title → lower the
    button `y` anchor (e.g. `1.3` → `1.15`).
  These are layout defaults that belong in the library method, not
  caller-side `.update_layout()` patches.
- **Prefer Polars over Pandas** for all data processing.

## Quick orientation
- `python/pdstools`: library code (polars-based data tooling, reports).
- `python/tests`: pytest suite.
- `python/docs`: Sphinx docs (with a Makefile).
- `reports/*.qmd`: Quarto report templates.
- `examples/`: notebooks and example content.

## Setup (local)
Use uv for env management, mirroring CI.

```bash
uv sync --extra tests
```

Optional extras by area:
- Healthcheck + reports: `uv sync --extra healthcheck --extra tests`
- Docs: `uv sync --extra docs --extra all`
- Pre-commit hooks: `uv sync --extra dev`

One-time hook install for any checkout that may be pushed:

```bash
uv run pre-commit install
```

## Build / Lint / Test commands

### Linting and formatting
Preferred: run the pre-commit hooks (uses ruff + ruff-format + nb-clean).

```bash
uv run pre-commit run --all-files
```

Before pushing a branch, this is mandatory even if you already ran narrower
checks while iterating.

If you want to run ruff directly:

```bash
uv run ruff check ./python
uv run ruff format ./python
```

### Tests
The full suite takes ~10–15 minutes locally. **Prefer narrow, targeted runs**
(single file, single function, or `-k` keyword filter) while iterating.
Only run the full suite when you're done with a logical chunk of work
or when you need broad confidence (e.g. before pushing a PR). CI re-runs
everything, so a final full local run is usually enough.

Full test suite (CI default, with coverage; skips heavy tests):

```bash
uv run pytest \
  --cov=./python/pdstools \
  --cov-report=xml \
  --cov-config=./python/tests/.coveragerc \
  --ignore=python/tests/healthcheck/test_healthcheck.py \
  --ignore=python/tests/explanations/test_explanations_report.py \
  --ignore=python/tests/healthcheck/test_batch_healthcheck.py \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pegasystems/pega-datascientist-tools](https://github.com/pegasystems/pega-datascientist-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
