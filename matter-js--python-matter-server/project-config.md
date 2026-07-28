---
trigger: always_on
description: Python Matter Server is an officially certified Software Component that provides Matter controller support. It serves as the foundation for Matter support in Home Assistant and other projects. The server implements a Matter Controller over WebSockets using the official Matter SDK.
---

# Python Matter Server

Python Matter Server is an officially certified Software Component that provides Matter controller support. It serves as the foundation for Matter support in Home Assistant and other projects. The server implements a Matter Controller over WebSockets using the official Matter SDK.

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Working Effectively

### Bootstrap and Setup
- **CRITICAL**: Matter Server requires Linux or macOS with specific IPv6 networking configuration. Windows/WSL is NOT supported.
- Set up the complete development environment:
  ```bash
  scripts/setup.sh
  ```
  - Creates Python virtual environment in `.venv/`
  - Installs Python dependencies including Matter SDK components
  - Installs pre-commit hooks for code quality
  - **TIMING**: Typically takes 3-5 minutes. NEVER CANCEL - wait up to 10 minutes for completion.
  - **NETWORK ISSUE**: If pip install fails with timeout errors (common with Matter SDK dependencies), this is due to network limitations, not code issues.

### Python Server Development
- **Always run the bootstrapping steps first before any Python development**
- Start the Matter server:
  ```bash
  # Basic server (info log level)
  python -m matter_server.server

  # Debug mode
  python -m matter_server.server --log-level debug

  # SDK debug mode
  python -m matter_server.server --log-level-sdk progress
  ```
- Create `/data` directory with proper permissions if it doesn't exist
- Server runs on port 5580 by default (WebSocket endpoint)
- Alternative entry point: `python main.py`

### Dashboard Development
- **Dashboard setup** (requires Python dependencies to be available):
  ```bash
  cd dashboard
  script/setup
  ```
  - Runs `npm install` (~15 seconds)
  - Generates descriptions file from Python source
  - **TIMING**: ~30 seconds total. NEVER CANCEL - set timeout to 2+ minutes.

- **Development server**:
  ```bash
  cd dashboard
  script/develop
  ```
  - Starts TypeScript compiler in watch mode
  - Starts development server on http://localhost:5010
  - Live reload for development changes
  - **TIMING**: Starts in ~5 seconds

- **Production build**:
  ```bash
  cd dashboard
  script/build
  ```
  - Builds optimized TypeScript/JavaScript bundle
  - Copies build to `matter_server/dashboard/` directory
  - **TIMING**: ~10 seconds. NEVER CANCEL - set timeout to 2+ minutes.

### Testing and Code Quality
- **Run complete test suite**:
  ```bash
  scripts/run-in-env.sh pytest --durations 10 --cov-report term-missing --cov=matter_server --cov-report=xml tests/
  ```
  - **TIMING**: Typically 2-3 minutes. NEVER CANCEL - set timeout to 10+ minutes.

- **Pre-commit validation** (REQUIRED before commits):
  ```bash
  SKIP=no-commit-to-branch pre-commit run --all-files
  ```
  - Runs ruff (linting + formatting), pylint, mypy, codespell, and other checks
  - **TIMING**: 1-2 minutes for all files. NEVER CANCEL - set timeout to 5+ minutes.

- **Individual linting tools**:
  ```bash
  scripts/run-in-env.sh ruff check --fix
  scripts/run-in-env.sh ruff format
  scripts/run-in-env.sh pylint matter_server/ tests/
  scripts/run-in-env.sh mypy
  ```

## Validation Scenarios

**ALWAYS manually validate changes using complete end-to-end scenarios:**

### Python Server Validation
1. Start the server: `python -m matter_server.server --log-level debug`
2. Verify WebSocket endpoint responds (server starts without errors)
3. Check dashboard is accessible if built: verify `matter_server/dashboard/` contains files
4. Test example client: `python scripts/example.py` (requires dependencies)

### Dashboard Validation
1. Build dashboard: `cd dashboard && script/build`
2. Verify build artifacts: check `matter_server/dashboard/js/` contains compiled JavaScript
3. Start development server: `cd dashboard && script/develop`
4. Access http://localhost:5010 and verify dashboard loads
5. Test WebSocket connection input (should prompt for server URL)

### Pre-commit Validation
**CRITICAL**: Always run before pushing changes or CI will fail:
```bash
SKIP=no-commit-to-branch pre-commit run --all-files
```

## Repository Structure

### Key Directories
```
matter_server/           # Main Python package
├── client/             # Matter client library
├── server/             # Matter server implementation
├── common/             # Shared utilities
└── dashboard/          # Built web dashboard (auto-generated)

dashboard/              # Dashboard source code (TypeScript/Lit)
├── src/               # TypeScript source files
├── script/            # Build and development scripts
└── public/            # Static assets

scripts/               # Development utilities
├── setup.sh          # Main environment setup
├── example.py        # Server/client example
└── generate_descriptions.py  # Generates dashboard type definitions

tests/                 # Test suite (pytest-based)
docs/                  # Documentation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matter-js/python-matter-server](https://github.com/matter-js/python-matter-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
