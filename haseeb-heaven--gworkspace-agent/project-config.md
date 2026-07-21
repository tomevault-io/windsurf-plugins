---
trigger: always_on
description: This file provides AI coding agents with the context, conventions, and constraints needed to work effectively in this repository. It complements the README by covering build steps, test commands, code style, and project-specific rules that are most useful to an automated contributor.
---

# AGENTS.md

This file provides AI coding agents with the context, conventions, and constraints needed to work effectively in this repository. It complements the README by covering build steps, test commands, code style, and project-specific rules that are most useful to an automated contributor.

Compatible with: OpenAI Codex, GitHub Copilot Agent, Google Jules, Devin, Cursor, Windsurf, Aider, Goose, Factory, Zed, Warp, and any agent that reads AGENTS.md.

---

## Project Overview

**GWorkspace Agent** is a LangGraph-powered AI assistant for Google Workspace. It accepts natural language commands and translates them into Google Workspace API calls (Gmail, Drive, Calendar, Docs, Sheets, Slides, Meet, Chat) using a multi-agent planning and execution pipeline.

- **Language:** Python 3.11
- **Core Framework:** LangGraph + LangChain
- **LLM Backend:** OpenAI-compatible API (configurable via `LLM_PROVIDER` / `LLM_MODEL`)
- **Interfaces:** CLI, Desktop GUI (Tkinter), Web UI (Gradio), Telegram Bot, Cloud Run (REST)
- **Deployment:** Google Cloud Run via Docker
- **Package root:** `gws_assistant/`

---

## Dev Environment Setup

```bash
# 1. Create and activate virtual environment
python -m venv .venv && source .venv/bin/activate

# 2. Install all dependencies
pip install -r requirements.txt
pip install -e .
pip install ruff mypy pytest pytest-cov

# 3. Copy environment config
cp .env.example .env
# Fill in LLM_API_KEY, LLM_PROVIDER, LLM_MODEL, and GWS_BINARY_PATH

# 4. Run setup wizard (first time only)
python -m gws_assistant --setup
```

### Required Environment Variables

| Variable | Required | Description |
|---|---|---|
| `LLM_API_KEY` | ✅ | API key for LLM provider |
| `LLM_PROVIDER` | ✅ | `openai`, `anthropic`, or `openrouter` |
| `LLM_MODEL` | ✅ | Primary model — must support tool-calling |
| `LLM_FALLBACK_MODEL` | ❌ | First fallback model |
| `LLM_FALLBACK_MODEL2` | ❌ | Second fallback model |
| `GWS_BINARY_PATH` | ✅* | Path to GWS CLI binary (*skipped in CI) |
| `DEFAULT_RECIPIENT_EMAIL` | ❌ | Default email for send operations |
| `APP_LOG_LEVEL` | ❌ | Logging verbosity (default: `INFO`) |
| `CI` | auto | Set by pipeline — disables binary path validation |

---

## Build & Run Commands

```bash
# CLI interface
python gws_cli.py

# Desktop GUI
python gws_gui.py

# Web UI (Gradio)
python gws_gui_web.py

# Telegram Bot
python gws_telegram.py

# Docker (production)
docker build -t gworkspace-agent . && docker run --env-file .env gworkspace-agent
```

---

## Testing Instructions

The CI plan lives in `.github/workflows/pipeline.yml`. All checks must pass before a PR is merged.

```bash
# Run unit tests (safe for CI — no GWS binary needed)
pytest tests/ \
  -m "not live_integration" \
  --ignore=tests/manual \
  --ignore=tests/test_live_integration.py \
  --cov=gws_assistant \
  --cov-fail-under=70 \
  -v

# Run integration tests (mocked GWS)
pytest tests/test_integration.py \
  -m "not live_integration" \
  --cov=gws_assistant.langgraph_workflow \
  --cov=gws_assistant.execution \
  --cov-fail-under=65 \
  -v

# Run live integration tests (requires real GWS binary + credentials)
pytest tests/test_live_integration.py -m live_integration -v
```

### Test Structure

```
tests/
├── test_unit_*.py           # Unit tests — fully mocked, no GWS binary needed
├── test_integration.py      # Integration tests — mocked GWS, real LangGraph workflow
├── test_hardening_policy.py # Security/hardening policy tests
├── test_live_integration.py # Live tests — requires real credentials
└── manual/                  # Manual scripts — excluded from CI
```

### Testing Conventions

- Fix failing tests by correcting **source code**, never by modifying test assertions
- Coverage thresholds are enforced: unit ≥ 70%, integration ≥ 65% — do not lower them
- Tag live tests with `@pytest.mark.live_integration`
- Do not hardcode `CI=true` inside tests; use `monkeypatch.setenv` / `monkeypatch.delenv`
- Tests that exercise non-CI validation must explicitly unset CI: `monkeypatch.delenv("CI", raising=False)`
- Add or update tests for every code change, even if not requested

---

## Code Style Guidelines

```bash
# Lint (must pass with zero errors)
ruff check .

# Type check
mypy gws_assistant --ignore-missing-imports

# Auto-fix safe lint issues
ruff check . --fix
```

- Follow PEP 8 with a 100-character line limit
- All public functions and classes must have docstrings
- Use type annotations on all function signatures
- Inter-agent data must use Pydantic models from `models.py` — never pass raw `dict` between layers
- Return type from any tool wrapper must be `ToolResult` (defined in `models.py`)

---

## Pre-commit Hooks

This project uses pre-commit hooks to prevent accidental commits of secret files (`.env`, `secrets*.json`, etc.) and to enforce code quality.

### Installation

```bash
# After installing dependencies from requirements.txt
pre-commit install
```

### What It Blocks

The pre-commit configuration (`.pre-commit-config.yaml`) includes:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haseeb-heaven/gworkspace-agent](https://github.com/haseeb-heaven/gworkspace-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
