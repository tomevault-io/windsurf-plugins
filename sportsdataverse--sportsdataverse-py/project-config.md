---
trigger: always_on
description: handles the `{resultSets:[{name,headers,rowSet}]}` envelope for both families
---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [sportsdataverse-py Copilot Instructions](#sportsdataverse-py-copilot-instructions)
  - [Project Context](#project-context)
  - [Repository Workflow](#repository-workflow)
  - [Commit Convention](#commit-convention)
  - [Code Style](#code-style)
  - [DataFrame Engine — Polars 1.x](#dataframe-engine--polars-1x)
  - [HTTP Layer](#http-layer)
  - [Module Naming](#module-naming)
  - [NFL — nflreadpy Parity](#nfl--nflreadpy-parity)
    - [NFL Cache + Config](#nfl-cache--config)
    - [NFL — `ep_wp` model application + EPA/WPA](#nfl--ep_wp-model-application--epawpa)
  - [CFB — `cfb_play_participants`](#cfb--cfb_play_participants)
  - [CFB — offline reprocess (`CFBPlayProcess`, 0.0.52+)](#cfb--offline-reprocess-cfbplayprocess-0052)
  - [Module Pattern (NEW modules)](#module-pattern-new-modules)
  - [Test Conventions](#test-conventions)
  - [Build & Development Commands](#build--development-commands)
  - [Common Pitfalls](#common-pitfalls)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# sportsdataverse-py Copilot Instructions

## Project Context

`sportsdataverse-py` is the Python sister to the SportsDataverse R packages
(`wehoop`, `hoopR`, `cfbfastR`, etc.). It provides tidy access to play-by-play,
box score, schedule, roster, and other sports data across the NBA (+ G-League),
WNBA, NFL, MLB, NHL, MBB (men's college basketball), WBB (women's college
basketball), CFB (college football), PWHL + 20 HockeyTech minor/junior hockey
leagues, college hockey (M/W), college baseball + softball, soccer, cricket,
UFL/XFL/CFL, and odds endpoints.

When there is any conflict between this file and repository contributor
docs, follow `CONTRIBUTING.md`, `CLAUDE.md`, and the current test suite
under `tests/` as the source of truth.

## Repository Workflow

- Use feature branches for changes.
- `main` is the default branch and release branch. It uses **polars 1.x**
  end-to-end, all packaging metadata lives in `pyproject.toml` (PEP 621),
  and **uv** is the canonical day-to-day package manager.
- A separate `0.36-live` branch carries pandas-based development with a
  set of CFB PBP bug fixes that are gradually being translated into the
  polars `main` branch. Do NOT merge `0.36-live` wholesale — it would
  undo the polars migration. Translate semantic fixes individually.
- For any change to exported functions, update tests and documentation in
  the same PR.

## Commit Convention

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
feat(wbb): add espn_wbb_team_roster() season-level scraper
fix(cfb): correct kneel-down classification in cfb_pbp parser
docs(contributing): document uv workflow and skip_if_no_live gate
chore(deps): bump polars to >=1.0,<2.0 + re-lock
```

**Important: Never include AI agents or assistants (e.g., Claude, Copilot,
Cursor, GPT, Gemini) as co-authors on commits.** Omit all `Co-Authored-By`
trailers referencing AI tools. This applies whether the change was
generated, refactored, or reviewed with AI assistance — the human author
is the sole attributable contributor.

## Code Style

- Follow PEP 8 with Ruff formatting (line-length 120, configured in
  `pyproject.toml [tool.ruff]`). Ruff also handles import sorting,
  pyupgrade, and unused-import removal — black, isort, pycln, and
  flake8 are NOT used directly. The standalone `isort` hook in
  `.pre-commit-config.yaml` runs only to inject `from __future__ import
  annotations` at the top of every Python file via `--add-import`.
- Lint, format, and type-check before committing:

  ```sh
  uv run ruff check --fix sportsdataverse/<your_module>.py
  uv run ruff format sportsdataverse/<your_module>.py
  uv run mypy sportsdataverse/<your_module>.py
  ```

- New modules: full type hints required (params + returns); legacy modules
  remain un-typed for now. Per-module strict mypy overrides live in a
  single `[[tool.mypy.overrides]] module = [...]` list in `pyproject.toml`
  — append your module's dotted path to that list rather than creating a
  new override block.
- Use `from __future__ import annotations` only when targeting py3.8 (not
  applicable here; floor is py3.9). Modern type syntax (`dict[str, X]`,
  `X | None`, `list[int]`) is allowed everywhere.

## DataFrame Engine — Polars 1.x

Runtime is pinned to `polars>=1.0,<2.0`. Use the modern API surface:

| Use this | Don't use this (0.18-era) |
|---|---|
| `df.group_by("col")` | `df.groupby("col")` |
| `df.with_row_index("name")` | `df.with_row_count("name")` |
| `expr.map_elements(f, return_dtype=...)` | `expr.apply(f)` |
| `pl.struct(*cols)` | `pl.struct([cols])` |
| `pl.read_csv(schema_overrides=)` | `pl.read_csv(dtypes=)` |
| `Series.scatter()` | `Series.set_at_idx()` |
| `pl.len()` | `pl.count()` |
| `df.join(..., how="full", coalesce=True)` | `df.join(..., how="outer")` |
| `s.cum_sum()` | `s.cumsum()` |
| `s.shift(n=k, fill_value=v)` | `s.shift_and_fill(periods=k, fill_value=v)` |

If you find yourself reaching for a 0.18-style API, treat it as a bug —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sportsdataverse/sportsdataverse-py](https://github.com/sportsdataverse/sportsdataverse-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
