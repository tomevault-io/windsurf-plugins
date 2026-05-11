---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**D-PC Messenger** is a privacy-first, peer-to-peer messaging platform enabling collaborative AI intelligence through secure sharing of personal contexts. The project implements a novel "transactional communication" paradigm with end-to-end encryption and no server-stored messages.

**Architecture:** Multi-package monorepo with Python backend services and Tauri + SvelteKit desktop frontend.

---

## Repository Structure

```
dpc-messenger/
├── dpc-protocol/         # Shared protocol library (LGPL v3)
├── dpc-client/
│   ├── core/             # Python backend service
│   └── ui/               # Tauri + SvelteKit frontend
├── dpc-hub/              # Federation Hub server (AGPL v3)
├── specs/                # Protocol specifications
└── docs/                 # Documentation
```

---

## Common Commands

### Client Development

**Backend (Python):**
```bash
cd dpc-client/core
uv sync                           # Install dependencies (default)
uv run python run_service.py      # Run backend service (ports 8888, 9999)
uv run pytest                     # Run tests
uv run pytest --cov=dpc_client_core  # Run with coverage
```

**Platform-Specific Dependencies (macOS Apple Silicon):**

The client supports GPU-accelerated Whisper transcription on Apple Silicon (M1/M2/M3/M4) via MLX, but this is **optional** and not installed by default.

```bash
cd dpc-client/core

# Install without MLX (default, lightweight)
uv sync

# Install with MLX support (enables GPU-accelerated offline transcription)
uv sync --extra mlx
```

**Technical Details:**
- **Dependencies**: `mlx>=0.4.0`, `mlx-whisper>=0.2.0`
- **Platform Markers**: `sys_platform == 'darwin' and platform_machine == 'arm64'`
- **Size Impact**: MLX packages add ~500MB to installation
- **Defined In**: `[project.optional-dependencies] mlx = ["mlx", "mlx-whisper"]`
- **Graceful Fallback**: If MLX not installed, client uses OpenAI-compatible API or skips transcription
- **Cross-Platform Safety**: `--extra mlx` flag is safely ignored on Windows/Linux (platform markers prevent installation)

**Why Optional?**
- Keeps default installation lightweight (~200MB vs ~700MB with MLX)
- Not all users need offline transcription (can use cloud-based Whisper via OpenAI API)
- Users can add MLX support later without reinstalling other dependencies

**Frontend (Tauri + SvelteKit):**
```bash
cd dpc-client/ui
npm install                       # Install dependencies
npm run dev                       # Vite dev server only (port 1420)
npm run tauri dev                 # Full Tauri development mode
npm run build                     # Build frontend
npm run tauri build               # Build production desktop app
npm run check                     # TypeScript type checking
```

**Build Outputs:**
- Windows: `dpc-client/ui/src-tauri/target/release/dpc-messenger.exe`
- Installers: `dpc-client/ui/src-tauri/target/release/bundle/`

### Hub Development

```bash
cd dpc-hub
docker-compose up -d              # Start PostgreSQL
cp .env.example .env              # Configure environment
uv sync                           # Install dependencies
uv run alembic upgrade head       # Run database migrations
uv run uvicorn dpc_hub.main:app --reload  # Start dev server

# Database migrations
uv run alembic revision --autogenerate -m "description"  # Create migration
uv run alembic upgrade head       # Apply migrations
uv run alembic downgrade -1       # Rollback last migration

# Testing and linting
uv run pytest                     # Run tests
uv run pytest --cov=dpc_hub       # Run with coverage
uv run black dpc_hub/             # Format code
uv run flake8 dpc_hub/            # Lint
uv run mypy dpc_hub/              # Type checking
```

### Protocol Library

```bash
cd dpc-protocol
uv sync                           # Install dependencies
uv run pytest                     # Run tests
```

### OAuth Configuration (Hub)

The Hub supports multiple OAuth providers for authentication:

**Supported Providers:**
- **Google OAuth** (required) - Primary authentication provider
- **GitHub OAuth** (optional) - Developer-friendly alternative

**Setup:**
1. Copy `.env.example` to `.env` in `dpc-hub/`
2. Add OAuth credentials:
   ```bash
   # Google OAuth (required)
   GOOGLE_CLIENT_ID="your_google_client_id"
   GOOGLE_CLIENT_SECRET="your_google_client_secret"

   # GitHub OAuth (optional)
   GITHUB_CLIENT_ID="your_github_client_id"
   GITHUB_CLIENT_SECRET="your_github_client_secret"
   ```

**Creating OAuth Apps:**
- **Google**: [Google Cloud Console](https://console.cloud.google.com/apis/credentials)
  - Callback URL: `http://localhost:8000/auth/google/callback` (dev)
- **GitHub**: [GitHub Developer Settings](https://github.com/settings/developers)
  - Callback URL: `http://localhost:8000/auth/github/callback` (dev)

**Client Usage (Backend):**
```python
# Login with Google (default)
await hub_client.login(provider="google")

# Login with GitHub
await hub_client.login(provider="github")
```

**Client UI:**
The UI displays two separate login buttons in the sidebar:
- **Google** button (blue) - Primary authentication

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikhashev/dpc-messenger](https://github.com/mikhashev/dpc-messenger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
