---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Prerequisites

- Python 3.11+
- Node.js 20+ (for UI development)
- Claude Code CLI

## Project Overview

This is an autonomous coding agent system with a React-based UI. It uses the Claude Agent SDK to build complete applications over multiple sessions using a two-agent pattern:

1. **Initializer Agent** - First session reads an app spec and creates features in a SQLite database
2. **Coding Agent** - Subsequent sessions implement features one by one, marking them as passing

## Commands

### npm Global Install (Recommended)

```bash
npm install -g autoforge-ai
autoforge                    # Start server (first run sets up Python venv)
autoforge config             # Edit ~/.autoforge/.env in $EDITOR
autoforge config --show      # Print active configuration
autoforge --port 9999        # Custom port
autoforge --no-browser       # Don't auto-open browser
autoforge --repair           # Delete and recreate ~/.autoforge/venv/
```

### From Source (Development)

```bash
# Launch Web UI (serves pre-built React app)
start_ui.bat      # Windows
./start_ui.sh     # macOS/Linux

# CLI menu
start.bat         # Windows
./start.sh        # macOS/Linux
```

### Python Backend (Manual)

```bash
# Create and activate virtual environment
python -m venv venv
venv\Scripts\activate  # Windows
source venv/bin/activate  # macOS/Linux

# Install dependencies
pip install -r requirements.txt

# Run the main CLI launcher
python start.py

# Run agent directly for a project (use absolute path or registered name)
python autonomous_agent_demo.py --project-dir C:/Projects/my-app
python autonomous_agent_demo.py --project-dir my-app  # if registered

# YOLO mode: rapid prototyping without browser testing
python autonomous_agent_demo.py --project-dir my-app --yolo

# Parallel mode: run multiple agents concurrently (1-5 agents)
python autonomous_agent_demo.py --project-dir my-app --parallel --max-concurrency 3

# Batch mode: implement multiple features per agent session (1-15)
python autonomous_agent_demo.py --project-dir my-app --batch-size 3

# Batch specific features by ID
python autonomous_agent_demo.py --project-dir my-app --batch-features 1,2,3
```

### YOLO Mode (Rapid Prototyping)

YOLO mode skips all testing for faster feature iteration:

```bash
# CLI
python autonomous_agent_demo.py --project-dir my-app --yolo

# UI: Toggle the lightning bolt button before starting the agent
```

**What's different in YOLO mode:**
- No regression testing
- No Playwright CLI (browser automation disabled)
- Features marked passing after lint/type-check succeeds
- Faster iteration for prototyping

**What's the same:**
- Lint and type-check still run to verify code compiles
- Feature MCP server for tracking progress
- All other development tools available

**When to use:** Early prototyping when you want to quickly scaffold features without verification overhead. Switch back to standard mode for production-quality development.

### React UI (in ui/ directory)

```bash
cd ui
npm install
npm run dev      # Development server (hot reload)
npm run build    # Production build (required for start_ui.bat)
npm run lint     # Run ESLint
```

**Note:** The `start_ui.bat` script serves the pre-built UI from `ui/dist/`. After making UI changes, run `npm run build` in the `ui/` directory.

## Testing

### Python

```bash
ruff check .                          # Lint
mypy .                                # Type check
python test_security.py               # Security unit tests (12 tests)
python test_security_integration.py   # Integration tests (9 tests)
python -m pytest test_client.py       # Client tests (20 tests)
python -m pytest test_dependency_resolver.py  # Dependency resolver tests (12 tests)
python -m pytest test_rate_limit_utils.py     # Rate limit tests (22 tests)
```

### React UI

```bash
cd ui
npm run lint          # ESLint
npm run build         # Type check + build (Vite 7)
npm run test:e2e      # Playwright end-to-end tests
npm run test:e2e:ui   # Playwright tests with UI
```

### CI/CD

GitHub Actions (`.github/workflows/ci.yml`) runs on push/PR to master:
- **Python job**: ruff lint + security tests
- **UI job**: ESLint + TypeScript build

### Code Quality

Configuration in `pyproject.toml`:
- ruff: Line length 120, Python 3.11 target
- mypy: Strict return type checking, ignores missing imports

## Architecture

### npm CLI (bin/, lib/)

The `autoforge` command is a Node.js wrapper that manages the Python environment and server lifecycle:
- `bin/autoforge.js` - Entry point (shebang script)
- `lib/cli.js` - Main CLI logic: Python 3.11+ detection (cross-platform), venv management at `~/.autoforge/venv/` with composite marker (requirements hash + Python version), `.env` config loading from `~/.autoforge/.env`, uvicorn server startup with PID file, and signal handling
- `package.json` - npm package config (`autoforge-ai` on npm), `files` whitelist with `__pycache__` exclusions, `prepublishOnly` builds the UI
- `requirements-prod.txt` - Runtime-only Python deps (excludes ruff, mypy, pytest)
- `.npmignore` - Excludes dev files, tests, UI source from the published tarball


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AutoForgeAI/autoforge](https://github.com/AutoForgeAI/autoforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
