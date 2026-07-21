---
trigger: always_on
description: [Streamlit](https://github.com/streamlit/streamlit) is an open-source (Apache 2.0) Python library for creating interactive web applications and dashboards with focus on data apps and internal tools.
---

# Streamlit Repo Overview

[Streamlit](https://github.com/streamlit/streamlit) is an open-source (Apache 2.0) Python library for creating interactive web applications and dashboards with focus on data apps and internal tools.

## Tech Stack

- **Backend (Server):** Python, Starlette/Uvicorn server, pytest
- **Frontend (Web UI):** TypeScript, React, Emotion (CSS-in-JS), Vite, Vitest
- **Communication:** Protocol Buffers (protobuf) over WebSocket.

## Folder Structure

- `lib/`: All backend code and assets.
  - `streamlit/`: The main Streamlit library package.
  - `streamlit/elements/`: Backend code of elements and widgets.
  - `streamlit/runtime/`: App runtime and execution logic.
  - `streamlit/web/`: Web server and CLI implementation
  - `tests`: Python unit tests (pytest).
- `frontend/`: All frontend code and assets.
  - `app/`: Streamlit application UI.
  - `lib/`: Shared TypeScript library that contains elements, widgets, and layouts.
  - `connection/`: WebSocket connection handling logic.
  - `utils/`: Shared utilities.
  - `component-lib/`: Library for building Streamlit custom components v1.
  - `component-v2-lib/`: Library for building Streamlit custom components v2.
- `proto/streamlit/proto/`: Protobuf definitions for client-server communication.
- `e2e_playwright/`: E2E tests using playwright (via pytest).
- `scripts/`: Utility scripts for development and CI/CD.
- `specs/`: Product and tech specs for Streamlit features.
- `.github/workflows/`: GitHub Actions workflows used for CI/CD.
- `wiki/`: Documentation relevant for development of Streamlit.

## Shell & Build Policy

- Prefer `make` targets for all dev tasks (tests, lint, format, builds).
- Always use `uv run` to run any Python command (e.g. `uv run streamlit`, `uv run pytest`, `uv run ruff`, `uv run mypy`, etc.).
- Always use `uv run` for git commands that trigger hooks (e.g. `uv run git commit`, `uv run git push`). Pre-commit hooks require the uv environment to run linters and formatters.
- For Python unit tests: `uv run pytest` commands are allowed and encouraged for running specific tests during development.
- For E2E tests: `uv run pytest` commands targeting `e2e_playwright/` files are blocked by policy.
  Use `make run-e2e-test <filename>` instead.

## `make` commands

Selection of `make` commands for development (run in the repo root):

- `help`: Show all available make commands. [~1s]
- `check`: Run all checks (format, lint, types, unit tests) on changed files only. Add `E2E_CHECK=true` to include E2E tests. [varies by changes]
- `protobuf`: Recompile Protobufs for Python and the frontend. [~5s]
- `autofix`: Autofix linting and formatting errors. [~30s]

**Backend Development (Python):**

- `python-lint`: Lint and check formatting of Python files (ruff). [~1s]
- `python-tests`: Run all Python unit tests (pytest). [~3min]
- `python-types`: Run the Python type checker (mypy & ty). [~30s]
- `python-format`: Format Python files (ruff). [~1s]

**Frontend Development (TypeScript):**

- `frontend-fast`: Build the frontend (vite). [~40s]
- `frontend-dev`: Start the frontend development server (hot-reload). [until stopped]
- `frontend-lint`: Lint and check formatting of all frontend files (oxlint + eslint). [~45s]
- `frontend-knip`: Run Knip dependency analysis. [~5s]
- `frontend-types`: Run the TypeScript type checker on all files (tsc). [~15s]
- `frontend-format`: Format all frontend files (oxfmt). [~2s]
- `frontend-tests`: Run all frontend unit tests (vitest). [~5min]

**E2E Testing (Playwright):**

- `run-e2e-test`: Run e2e test, via: `make run-e2e-test st_command_test.py`. [varies by test]

**Debugging backend & frontend:**

- `debug`: Start Streamlit backend and Vite dev server together, via: `make debug my_app.py`. [until stopped]
  - Frontend hot-reload: Changes to frontend code (`frontend/`) are applied within seconds.
  - Backend hot-reload: Only changes to the **app script** trigger a rerun. Changes to the Streamlit library itself (`lib/streamlit/`) require restarting `make debug`.
  - Logs are written to a per-session directory under `work-tmp/debug/` (e.g. `work-tmp/debug/<session>/backend.log` and `work-tmp/debug/<session>/frontend.log`).
  - `work-tmp/debug/latest/` is a symlink to the most recent debug session (e.g. `work-tmp/debug/latest/backend.log`). If multiple sessions are running simultaneously, this symlink can move—prefer using the session directory printed by `make debug`.
  - Log files are cleared at the start of each session and persist after exit for post-mortem analysis.
  - Browser `console.log()` output appears in the session’s `frontend.log`.
  - See [.claude/skills/debugging-streamlit/SKILL.md](.claude/skills/debugging-streamlit/SKILL.md) for the full debugging guide.

### Development Tips

- **Follow existing patterns**: Check neighboring files for conventions.
- You can use the `work-tmp` directory to store temporary files, specs, and scripts.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [streamlit/streamlit](https://github.com/streamlit/streamlit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
