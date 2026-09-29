---
trigger: always_on
description: Chrys is a Python 3.14+ agent platform: Textual TUI, headless CLI and ACP server. Users see it as **iCode** / `icode` (`foundation/branding.py`; `msg()` fallbacks write `{app}`, bound with `app=APP_DISPLAY_NAME`); the package, the `chrys` command, `~/.chrys`, `CHRYS_*`, wire/header names and data keys keep `chrys`. `pyproject.toml` is the source of truth for version and deps. Paths are under `src/chrys/` unless they start with `tests/`, `scripts/`, `docs/`, `locales/`, `.github/` or name a root 
---

# AGENTS.md

Chrys is a Python 3.14+ agent platform: Textual TUI, headless CLI and ACP server. Users see it as **iCode** / `icode` (`foundation/branding.py`; `msg()` fallbacks write `{app}`, bound with `app=APP_DISPLAY_NAME`); the package, the `chrys` command, `~/.chrys`, `CHRYS_*`, wire/header names and data keys keep `chrys`. `pyproject.toml` is the source of truth for version and deps. Paths are under `src/chrys/` unless they start with `tests/`, `scripts/`, `docs/`, `locales/`, `.github/` or name a root file; a bare file name continues the directory named just before it.

## Commands
```bash
uv sync --extra all                  # setup, as CI; bare `uv sync` uninstalls the extras
./scripts/fetch_rg.sh                # vendored ripgrep, gitignored (Windows: .ps1); search falls back to rg on PATH
uv run icode                         # TUI (-s session, -a agent, -m model, -C workdir); `chrys` is the same entry point
uv run icode run "<prompt>" -a Code  # headless, approval BYPASS (as `icode workflow run`)
uv run icode acp                     # ACP stdio server; also: serve, agents, models, workflow, trajectory, install
uv run python scripts/chrys_test.py --smart --paths <changed files...>   # default verification
uv run python scripts/chrys_test.py --full        # complete non-integration suite; only when asked
uv run pytest tests/x/test_y.py::test_fn -n0      # debugging only
uv run ruff check src/ tests/ scripts/chrys_test.py scripts/calibrate_gc_freeze.py scripts/gc_freeze_calibration_math.py   # pyproject's fix = true EDITS files (--no-fix inspects); CI auto-fixes too, so only unfixable errors fail there
uv run ruff format --check src/ tests/ scripts/chrys_test.py scripts/calibrate_gc_freeze.py scripts/gc_freeze_calibration_math.py
uv run ty check --error-on-warning --python-platform linux src/chrys   # repeat for darwin and win32; the src/chrys argument is required
uv run python scripts/i18n.py extract   # then update, translate, compile, check (see i18n)
```
- Run every project tool through `uv run`; global installs are other versions than the pins.
- Direct pytest also collects `integration` and `gc_calibration` tests (add `-m "not integration and not gc_calibration"` for directory runs), and under the default `-n 8` a mistyped path exits 5 "no tests ran" instead of erroring: `ls` it first.

## Verification, gates & commits
- Verify with Smart Test, passing `--paths` = only the files this task changed (include deleted paths and both sides of a rename); without `--paths` it also picks up every branch and dirty-tree change. Don't hand-compute the affected scope. It escalates to the full suite by itself on `pyproject.toml`, `uv.lock`, `.python-version`, root/`tests/` conftest or pytest-config changes; otherwise run `--full` only when asked.
- Smart Test is a local optimization only: ≤2 import hops, fixture consumers and explicit watches; distant consumers are left to CI. CI/CD keeps its direct complete PR gates and must never use Smart Test to select or skip tests.
- A test that reads repo files without importing them (subprocess fixture, file scan) needs a `REGULAR_RULES`/`ARCHITECTURE_RULES` watch in `scripts/chrys_test.py`; otherwise edits to those files select it only by the nearby-directory fallback, if at all.
- Ruff, format, ty, i18n and `uv build` are separate gates. Say which suite ran; a Smart Test pass is never a full-suite pass. Python-3.9 workflow-worker cases skip without a 3.9 interpreter (`CHRYS_PY39_INTERPRETER`, PATH or Homebrew keg); Linux/Windows CI runs them.
- CI (`.github/workflows/ci.yml`; partitions pinned by `tests/architecture/test_ci_test_partitions.py`) runs `-m "not integration and not gc_calibration"` in per-OS core/TUI shards and `tests/architecture` alone at `-n 0`. Decorate OS-independent whole-repo scans with `@CI_LINUX_ONLY` (`tests/support/ci.py`).
- Refresh generated artifacts in the same change: i18n catalogs (see i18n); after editing a builtin workflow template, `uv run python -m tests.support.workflow_builtins`; when an async wait (`await`, `async with`/`for`) under foundation/kernel/service/orchestration is added, removed, reordered or edited (line moves alone don't count), `uv run python -m tests.support.trajectory_wait_inventory` for `tests/architecture/trajectory_wait_manifest.json` — it keeps a reviewed case only while its `module:qualname:ordinal`, expression and primitive match, so adding or removing a wait also reopens later waits in that function; review every new fail-closed `B` case.
- User-visible change → update the affected `docs/en/` and `docs/zh-Hans/` pages in the same change (see User guide).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openJiuwen-ai/iCode](https://github.com/openJiuwen-ai/iCode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
