---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An "OS setup" project with two parts:
1. A cross-platform Python **terminal UI app** (`personal-os-setup`, package `src/personal_os_setup/`) that detects the host OS/distro and lets the user install packages and run system actions (WSL management, zsh/oh-my-zsh, NVIDIA drivers, Windows Terminal config, chezmoi dotfiles, Docker post-install, etc).
2. A **documentation hub** (`docs/`) covering Windows/WSL2, Linux, macOS, Android TV, and home-server setups, published as a static site via `properdocs`/`mkdocs`.

`src/awesome_os/` only contains stale `__pycache__` directories from a prior package name — the real package is `personal_os_setup`. Don't treat it as live code.

## Commands

All common tasks go through `make` (backed by `makefiles/*.mk`, included from the root `Makefile`). Run `make help` to list targets by category.

- `make install-dev` — installs uv (if needed) and all dependency groups (dev + docs) into `.venv`. Use this, not `uv pip install -e .` unless doing something one-off.
- `make test` — `uv run pytest tests/unit` (exit code 5 / "no tests collected" is treated as success).
- `make test-integration` — runs `tests/integration/*` (requires Ubuntu with passwordless sudo; not runnable on macOS/dev machines).
- Run a single test: `uv run pytest tests/unit/test_detect_os.py::test_name -v`
- `make pre-commit` — installs hooks then `pre-commit run --all-files` (ruff check+format, detect-secrets, commitizen, check-yaml/json/toml, uv-lock sync, etc). Run this and `make test` before opening a PR.
- `make lint` / `make format` — `ruff check .` / `ruff format .` directly.
- `make run` — launches the TUI (`python src/personal_os_setup/frontend/main.py`).
- `make deploy-doc-local` — builds and serves the `properdocs`/mkdocs site locally.
- `make build-package` — `uv build` (wheel).
- `make install-act` / `make act` — run GitHub Actions locally via `act` (requires Docker).

CI runs the identical `make pre-commit` / `make test` commands used locally — there is no separate GH-Actions-only tooling, so if it passes locally it passes in CI.

## Architecture

### OS/distro detection → package catalog → UI

`detect_os.py` is the entry point for environment awareness:
- `detect_os()` returns an `OSInfo(family, distro, info)` — family is `windows`/`darwin`/`linux`/`unknown`; on Linux, `distro` comes from `/etc/os-release` `ID` (e.g. `ubuntu`, `cachyos`), and WSL is detected separately (`_is_wsl()`, checks `/proc/sys/fs/binfmt_misc/WSLInterop` or `/proc/version`).
- `build_packages_for_os()` loads `src/personal_os_setup/config/packages.yaml` (the single source of truth for installable packages, keyed by distro → manager → category → package list) and filters it to the current distro via `PackageCatalog.for_distro()`, producing a flat list of `PackageRef(name, manager, category)`.

### Package managers (`tasks/managers/`)

Each backend (apt, snap, brew/cask, winget, msstore, pacman/paru, webinstall) implements the `PackageManager` protocol in `tasks/managers/base.py`: `is_installed`, `install`, `update`, `upgrade`, `cleanup`. `tasks/factory.py` maps `(distro, manager_name)` → manager class in `_PACKAGE_MANAGER_FACTORY_BY_DISTRO`, and `_PRIMARY_MANAGERS_BY_DISTRO` controls which managers are surfaced in the UI per distro (e.g. only `apt` shown for Ubuntu even though `snap`/`webinstall` also exist) to avoid duplicate buttons.

`tasks/managers/_shared.py` centralizes result-construction boilerplate reused across backends: `command_details()`/`format_failed_command()` (stdout/stderr → details text), `sudo_required_task_result()`/`sudo_required_install_result()`, `missing_executable_task_result()`/`missing_executable_install_result()`, and `winget_path()`/`winget_list_shows_installed()` (shared by the winget and msstore backends). Each manager still calls `shutil.which(...)`/`sudo_non_interactive_ok()` itself (not through `_shared.py`) so unit tests can keep patching those checks at the manager's own module path — `_shared.py` only builds the resulting `TaskResult`/`InstallResult` once the check has failed. Follow this pattern (local check, shared result-builder) when adding a new manager rather than re-deriving the boilerplate.

### System actions (`tasks/factory.py::get_system_action_sections`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AmineDjeghri/awesome-os-setup](https://github.com/AmineDjeghri/awesome-os-setup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
