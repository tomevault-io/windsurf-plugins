---
trigger: always_on
description: This file provides guidance to coding agents (Claude Code, and others that read `AGENTS.md`) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents (Claude Code, and others that read `AGENTS.md`) when working with code in this repository.

## What this is

Harlequin is a SQL IDE that runs in the terminal, built on [Textual](https://textual.textualize.io/). It connects to databases through pluggable **adapters**; this repo contains the core app plus the two adapters that ship with it (DuckDB, SQLite) and one keymap plugin (VS Code).

User-facing docs live in a separate repo, [`tconbeer/harlequin-web`](https://github.com/tconbeer/harlequin-web) (published at harlequin.sh). Doc changes for a feature go there, not here.

## We own most of the stack — fix things upstream

Much of what Harlequin depends on is maintained in the same org, so a limitation in a dependency is usually not something to work around:

- **`textual-fastdatatable`** and **`textual-textarea`**, the component libraries the Results Viewer and Query Editor are built on.
- **`pytest-textual-snapshot`**, pinned to a fork (see the `test` dependency group).
- **Several adapters** — `harlequin_duckdb` and `harlequin_sqlite` in this repo, plus out-of-tree ones like `harlequin-postgres` and `harlequin-mysql`.

**When the real fix belongs upstream, make it upstream.** A workaround here is a permanent tax on this repo for a problem whose actual home is one release away, and it will be read by the next person as intended design. If you can't reach the upstream repo yourself, write the change up as an issue and hand it over — don't quietly absorb it.

Worked example: `create_backend()` had no way to accept the column names a cursor reported, so a result with no rows arrived with no header. The reconciliation that would otherwise have lived in `harlequin.query` forever — special cases for `None`, for an empty sequence, for `f0`/`f1` names, for a count mismatch — became a `column_names` argument in textual-fastdatatable 0.17.0 and one line here.

The costs are real and worth planning around rather than avoiding: an upstream fix needs a release and a pin bump before this repo sees it, and a component-library bump can bring snapshot churn. Neither outweighs owning a workaround forever.

## Commands

Everything runs through `uv`. `make check` is the full pre-PR loop; run it before pushing.

```bash
make check      # sync deps, ruff format+fix, pytest (3.10 + 3.12 py12 tests), mypy
make lint       # ruff format + ruff check --fix + mypy only
make serve      # run the app against ./f1.db in textual dev mode
make sqlite     # run the app with the sqlite adapter, dev mode
make keys       # run the `harlequin --keys` keymap editor, dev mode
```

Individual tools:

```bash
uv run pytest -m "not online"                      # the standard test run
uv run pytest tests/functional_tests/test_app.py::test_select_1 -n0   # single test
uv run mypy                                        # strict; covers src/ and tests/
uv run ruff format . && uv run ruff check . --fix
uv run harlequin [OPTIONS] [CONN_STR]              # run the CLI from source
```

- Tests run under xdist (`-n auto` is in `addopts`). Pass `-n0` when using a debugger or `-s`.
- Markers: `online` (needs network + secrets; always deselect locally), `py12` (only runs on 3.12+), `use_cache` (opts a test back into the buffer/catalog caches that an autouse fixture otherwise disables).
- `TEST_MARKERS` in the Makefile is a marker *expression*, not a flag — a second `-m` silently overrides the first.

## Testing notes

Functional tests drive the real Textual app via `pilot` and assert on both messages and SVG snapshots. Unit tests use syrupy directly for the headless output formats (`tests/unit_tests/test_golden_formats.py`), as single-file binary snapshots — same `--snapshot-update` workflow, and `.gitattributes` pins every `__snapshots__` file to LF so a Windows checkout can't rewrite what they assert.

- **A snapshot mismatch is not automatically a failure.** What matters is whether the test passes. Several tests deliberately skip their snapshot assertion (e.g. the `transaction_button_visible` fixture, because SQLite on 3.12+ grows a transaction button that isn't in the baseline), and CI runs with `--snapshot-warn-unused`.
- Snapshots are committed from **Python 3.10**, the lowest supported version. Regenerating requires two runs, and `tests/conftest.py::pytest_configure` will refuse a run that would clobber the baseline:

```bash
uv run pytest --snapshot-update                                                   # on 3.10
uv run --python 3.12 --group test pytest -m 'py12 and not online' --snapshot-update  # py12-only snaps
```

- Async tests need `@pytest.mark.asyncio`. Await `wait_for_workers(app)` (fixture) rather than sleeping — it skips the catalog background loader, which never finishes on its own.
- Shared fixtures: `tests/conftest.py` builds throwaway DuckDB/SQLite databases (`tiny_*`, `small_*`) and app instances (`app`, `app_all_adapters`, `app_small_duck`, …); `app_all_adapters` is parametrized so one test body covers both bundled adapters.

## Import hygiene


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tconbeer/harlequin](https://github.com/tconbeer/harlequin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
