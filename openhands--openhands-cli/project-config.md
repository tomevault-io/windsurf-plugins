---
trigger: always_on
description: OpenHands CLI is a standalone terminal interface (Textual TUI) for interacting with the OpenHands agent.
---

# Repository Guidelines

## Repository Purpose
OpenHands CLI is a standalone terminal interface (Textual TUI) for interacting with the OpenHands agent.

This repo contains the current CLI UX, including the Textual TUI and a browser-served view via `openhands web`.


### References
- Agent-sdk example: https://github.com/All-Hands-AI/agent-sdk/blob/main/examples/hello_world.py
- If you need to compare with upstream OpenHands code, use `$GITHUB_TOKEN` for access.

## Project Structure & Module Organization
- `openhands_cli/`: Core CLI/TUI code (`openhands_cli/entrypoint.py`, `openhands_cli/tui/`, `openhands_cli/auth/`, `openhands_cli/mcp/`, `openhands_cli/cloud/`, `openhands_cli/user_actions/`, `openhands_cli/conversations/`, `openhands_cli/theme.py`, helpers in `openhands_cli/utils.py`). Keep new modules snake_case and colocate tests.
- `tests/`: Pytest suite covering units, integration, and snapshot tests; mirrors source layout. `tui_e2e/`: tests for the PyInstaller-built executable.
- `scripts/acp/`: JSON-RPC and debug helpers for ACP development; `hooks/`: PyInstaller/runtime hooks.
- Tooling & packaging: `Makefile` for common tasks, `build.sh`/`build.py` for PyInstaller artifacts, `openhands-cli.spec` for the frozen binary, `uv.lock` for resolved deps.
- `.agents/skills/`: agent guidance for this repo.

## Setup, Build, and Development Commands
This repository uses **uv** for dependency management and running tooling (such as in `Makefile`, CI workflows, and `uv.lock`). Use `uv` 0.11.6 or newer for local development; older versions can serialize `uv.lock` differently around relative `exclude-newer`. Avoid using `pip install ...` directly if possible.

- minimum supported `uv` version: `0.11.6`
- install dependencies: `make install` (runs `uv sync`)
- install dev dependencies: `make install-dev` (runs `uv sync --group dev`)
- install pre-commit hooks: `uv run pre-commit install` (included in `make build`)
- build (sync + install hooks): `make build`
- lint (all pre-commit hooks): `make lint`
- format: `make format`
- run the Textual TUI (interactive; prefer running inside tmux so you can detach with `Ctrl+b d`): `make run` (or `uv run openhands`)
- run the Textual TUI (automation-friendly; use for agent-driven runs): `uv run openhands --exit-without-confirmation` (quit with `Ctrl+Q`; `Ctrl+C` does not work once the TUI is running)
- **fast TUI development** (see [Fast TUI Development Workflow](#fast-tui-development-workflow) below):
  - `make run-watch` - Auto-restart on file changes (recommended for most development)

- run the browser-served web app (Textual `textual-serve`): `openhands web`
- run the Docker-based OpenHands GUI server: `openhands serve`
- run the ACP entrypoint: `uv run openhands-acp`
- run unit/integration tests: `make test` (for faster runs: `uv run pytest -m "not integration" --ignore=tests/snapshots`)
- run snapshot tests (Textual UI): `make test-snapshots` (or `uv run pytest tests/snapshots -v`; use `--snapshot-update` when updating snapshots)
- run binary tests: `make test-binary` (or `uv run pytest tui_e2e`)
- run unit/integration + snapshot tests together: `make test-all`
- build PyInstaller binaries: `./build.sh --install-pyinstaller`

## Development Guidelines

### Fast TUI Development Workflow

#### Auto-restart on file changes (recommended)

The fastest way to iterate on TUI changes:

```bash
make run-watch
```

This watches `openhands_cli/` and automatically restarts the app when you save any `.py` or `.tcss` file. Just edit, save, and see your changes.

### Linting Requirements
**Before any commit, run `make lint` and only commit after it passes.** Use `make lint` to run all pre-commit hooks on all files, and do it before every commit (not after) to avoid CI failures.

### Typing Requirements
Prefer modern typing syntax (`X | None` over `Optional[X]`) in new code.

### Documentation Guidelines
- Don’t add new root-level `.md` files or “summary updates” to `README.md` unless explicitly requested (use this `AGENTS.md` for repo guidance).

## Coding Style & Naming Conventions
- Python 3.12, ruff formatting (88-char line limit, double quotes).
- Ruff enforced rules: pycodestyle, pyflakes, isort, pyupgrade, unused-arg checks (tests allow fixture-style args), and guards against mutable defaults.
- Keep modules/dirs snake_case; classes in CapWords; user-facing commands/flags kebab-case as in existing entrypoints.
- Type checking via `pyright` (`uv run pyright`); prefer type hints on new functions and public interfaces.

## Testing Guidelines
- Unit/integration tests live under `tests/` (excluding `tests/snapshots`) and run via `make test`.
- Snapshot tests live under `tests/snapshots/` and run via `make test-snapshots`.
- Binary tests live under `tui_e2e/` and run via `make test-binary`.
- Pytest discovery: files `test_*.py`, classes `Test*`, functions `test_*`. Use `@pytest.mark.integration` for costly flows.
- Match test locations to implementation (`tests/` mirrors `openhands_cli/`); add fixtures in `tests/conftest.py` when shared.
- Run `make test` before PRs; run snapshot/binary tests when relevant to the change.

### Binary Tests with Mock LLM

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenHands/OpenHands-CLI](https://github.com/OpenHands/OpenHands-CLI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
