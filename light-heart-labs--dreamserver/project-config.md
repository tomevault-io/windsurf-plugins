---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dream Server is a fully local AI stack (LLM inference, chat, voice, agents, workflows, RAG, image generation, privacy tools) deployed on user hardware with a single command. It supports Linux (NVIDIA + AMD), Windows (WSL2), and macOS (Apple Silicon). The project is primarily Bash (installer/CLI), Python (dashboard-api, services), and React/Vite (dashboard UI).

## Repository Structure

The repo has two layers:

- **Root level** — outer wrapper with top-level README, install scripts (`install.sh`, `install.ps1`), CI workflows (`.github/workflows/`), and `resources/` (cookbooks, blog, dev tools, frameworks)
- **`dream-server/`** — the core product containing all deployable code

Within `dream-server/`:

- **`install-core.sh`** — thin orchestrator (~184 lines) that sources libs then runs phases in order
- **`installers/lib/`** — pure function libraries (constants, logging, UI, GPU detection, tier mapping, packaging, compose selection)
- **`installers/phases/`** — 13 sequential install steps (`01-preflight` through `13-summary`), each sourced by install-core
- **`installers/macos/`**, **`installers/windows/`** — platform-specific installer variants
- **`extensions/services/`** — 17 service extensions, each a directory with `manifest.yaml` + optional `compose.yaml` and GPU overlays
- **`docker-compose.base.yml`** — core service definitions; `docker-compose.{amd,nvidia,apple}.yml` are GPU overlays
- **`dream-cli`** — main CLI tool (~45K lines Bash) for managing the stack
- **`config/`** — backend configs (`backends/amd.json`, `nvidia.json`, etc.), GPU database, LiteLLM config, hardware classes
- **`extensions/services/dashboard-api/`** — Python FastAPI backend (with `routers/`, `tests/`)
- **`extensions/services/dashboard/`** — React + Vite + Tailwind frontend (`src/`)
- **`scripts/`** — operational scripts (health checks, model management, compose stack resolution, doctor, preflight)
- **`tests/`** — shell-based tests (tier map, contracts, smoke tests, integration)
- **`lib/`** — shared Bash utilities (safe-env, service-registry, progress, QR code)

## Build & Development Commands

All commands run from `dream-server/` directory unless noted.

### Linting and Validation

```bash
make lint          # Shell syntax check (bash -n) + Python compile check
make test          # Tier map tests + installer contract tests + preflight fixtures
make smoke         # Platform smoke tests (linux-amd, linux-nvidia, wsl, macos)
make simulate      # Installer simulation harness
make gate          # Full pre-release: lint + test + smoke + simulate
make doctor        # Run diagnostic report
```

### Running a Single Test

```bash
bash tests/test-tier-map.sh                      # Tier mapping tests
bash tests/contracts/test-installer-contracts.sh  # Installer contracts
bash tests/contracts/test-preflight-fixtures.sh   # Preflight fixtures
bash tests/smoke/linux-nvidia.sh                  # Single smoke test
```

### Dashboard API (Python/FastAPI)

```bash
cd extensions/services/dashboard-api
pytest tests/                    # Run all dashboard-api tests
pytest tests/test_routers.py     # Run a specific test file
```

### Dashboard UI (React/Vite)

```bash
cd extensions/services/dashboard
npm install
npm run dev      # Dev server
npm run build    # Production build
npm run lint     # ESLint
```

### Pre-commit Hooks

The root `.pre-commit-config.yaml` runs gitleaks (secret scanning), private key detection, and large file checks. Install with:
```bash
pip install pre-commit && pre-commit install
```

## CI Workflows

GitHub Actions in `.github/workflows/`:
- **lint-shell.yml** — ShellCheck on all `.sh` files
- **lint-python.yml** — Python linting
- **type-check-python.yml** — Python type checking
- **dashboard.yml** — Dashboard build/lint
- **test-linux.yml** — Linux test suite + installer simulation (uploads artifacts)
- **matrix-smoke.yml** — Multi-distro smoke tests (6 distros)
- **validate-compose.yml** — Docker Compose validation
- **secret-scan.yml** — Secret scanning
- **lint-powershell.yml** — PowerShell linting for Windows installer

## Architecture Key Concepts

### Installer Architecture

The installer is modular with a strict separation: `installers/lib/` contains pure functions (no side effects), `installers/phases/` contain sequential steps that execute on `source`. Every module has a standardized header (Purpose, Expects, Provides, Modder notes). The orchestrator (`install-core.sh`) sets `INSTALL_PHASE` before each phase for error reporting.

### Extension System

Every service is an extension under `extensions/services/<name>/`. Each has a `manifest.yaml` defining metadata (id, port, health endpoint, container name, aliases, category, GPU backends, feature flags). Extensions with `compose.yaml` get auto-merged into the Docker Compose stack by `scripts/resolve-compose-stack.sh`. Core services (llama-server, open-webui, dashboard, dashboard-api) only have manifests — their compose lives in `docker-compose.base.yml`.

### GPU Backend / Tier System


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Light-Heart-Labs/DreamServer](https://github.com/Light-Heart-Labs/DreamServer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
