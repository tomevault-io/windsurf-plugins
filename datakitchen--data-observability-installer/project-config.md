---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This repo ships **one artifact**: `dk-installer.py` — a single-file, stdlib-only Python script that end users download and run to install/upgrade/start/demo the open-source DataKitchen products (TestGen and DataOps Observability) locally. TestGen supports both Docker Compose and pip-via-uv install modes; Observability is Docker-only. On Windows it is also packaged as `dk-installer.exe` via PyInstaller (see `.github/workflows/release_exe.yml`).

The `demo/` directory is a separate deliverable: it is built into the `datakitchen/data-observability-demo` Docker image that `dk-installer.py` pulls at runtime to generate demo data. It is **not** imported by the installer.

## Common commands

```bash
pip install .[dev,test]                 # install ruff + pytest (project has no runtime deps)

ruff check --show-fixes                 # lint (CI-enforced)
ruff format --check --diff              # format check (CI-enforced)
ruff format                             # apply formatting

pytest                                  # run full test suite
pytest tests/test_tg_install.py         # single file
pytest tests/test_action.py::test_name  # single test
pytest -m unit                          # only unit-marked tests
pytest -m integration                   # only integration-marked tests
pytest --cov --cov-report=term-missing  # with coverage (matches CI)

python3 dk-installer.py --help          # see installer CLI
python3 dk-installer.py tg install      # run an action locally during dev
```

Building the Windows `.exe` happens automatically on every push to `main` (`release_exe.yml` → PyInstaller → GitHub Release tagged `latest`). For local builds on Windows, see `docs/build_windows_installer.md`.

## Architecture: how `dk-installer.py` is organized

The installer is a ~2300-line single file intentionally using only the Python stdlib — users run it without installing any packages. Do not introduce third-party runtime dependencies.

Core abstractions (all in `dk-installer.py`):

- **`Installer`** — top-level argparse wrapper. `get_installer_instance()` at the bottom of the file registers the two products (`obs`, `tg`) and their actions. Each product sets compose-file defaults (`compose_file_name`, `compose_project_name`) that flow into actions via argparse `set_defaults`.
- **`Action`** — base class for one CLI subcommand (e.g., `tg install`). Owns session-scoped concerns: creates a timestamped log folder under `.dk-installer/` (or `%LOCALAPPDATA%/DataKitchenApps/` on Windows), configures logging, zips logs on exit, wraps execution in `AnalyticsWrapper`, enforces `requirements` (list of `Requirement` objects that shell out to check `docker`, `docker compose`, etc.), and provides `run_cmd` / `run_cmd_retries` — always use these rather than raw `subprocess` so output is captured per-command into the session zip.
- **`MultiStepAction`** — `Action` subclass that declares a `steps: list[type[Step]]`. Each `Step` has `pre_execute` (run for all steps before any executes — validation phase) then `execute` (the actual work). On any step failure, remaining steps are skipped and `on_action_fail` runs in reverse order; on success, `on_action_success` runs in reverse order. **Most install/upgrade actions are `MultiStepAction`s** — when adding a new install phase, write a new `Step` class and add it to the list.
- **`Step`** — unit of work inside a `MultiStepAction`. Steps share state via `action.ctx` (a dict on the parent action). Raising `SkipStep` from `execute` marks it SKIPPED; raising any other exception marks it FAILED and aborts the action if `required = True`.
- **`ComposeActionMixin` / `ComposeDeleteAction` / `ComposePullImagesStep` / `ComposeStartStep` / `CreateComposeFileStepBase`** — shared building blocks for both products. `Obs*` and `TestGen*` classes specialize these.
- **`AnalyticsWrapper`** — sends anonymous Mixpanel events for each action (disabled with `--no-analytics` or `DK_INSTALLER_ANALYTICS=no`). Instance ID is persisted to `.dk-installer/instance.txt`. Don't log PII here.
- **`Console`** (global `CONSOLE`) — all user-facing output goes through this; don't use bare `print` for user messages (the menu code and `collect_user_input` are the exceptions).
- **`Menu`** / `show_menu` — only used when the frozen Windows `.exe` is launched with no arguments (double-click). Not part of the CLI flow on Unix.

The action registry in `get_installer_instance()` is the authoritative list of user-facing commands — to add a new command, add an `Action` subclass there.

### TestGen install modes

TestGen has two install modes: `docker` (Compose) and `pip` (uv-managed venv with embedded Postgres). Mode is recorded at install time in a JSON marker file (`dk-tg-install.json`) so `tg upgrade` / `tg delete` / `tg start` / `tg run-demo` / `tg delete-demo` know which path to take.

The five `Testgen*Action` classes that span both modes follow a unified pattern:
- `_per_invocation_attrs` includes `_resolved_mode` (and `steps` / `intro_text` for the `MultiStepAction`-based ones) so menu re-runs start clean.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DataKitchen/data-observability-installer](https://github.com/DataKitchen/data-observability-installer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
