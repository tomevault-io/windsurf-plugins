---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an AI Agent Platform that enables users to create, manage, and chat with customizable AI agents powered by **Claude Agent SDK**. It is a desktop application built with Tauri 2.0 + React + Python FastAPI, using SQLite for storage and local filesystem + Git for skill management.

The **desktop version** (`desktop/` directory) is the primary development target and supports **Windows, macOS, and Linux**.

## Development Commands

### Desktop Development (Primary)

```bash
cd desktop

# Install dependencies
npm install

# Configure environment (required: set ANTHROPIC_API_KEY)
cp backend.env.example ../backend/.env
# Edit ../backend/.env and add your API key

# Development mode (hot reload)
npm run tauri:dev

# Build production
npm run build:all      # Full build: backend + frontend + Tauri
npm run build:backend  # Only Python backend (PyInstaller)
npm run tauri:build    # Only Tauri app (requires backend built first)

# Testing
npm run test           # Watch mode
npm run test:run       # Single run
npm run test -- src/components/Button.test.tsx  # Run specific test file
npm run lint
```

### Backend Development

```bash
cd backend

# Setup (using uv - recommended)
uv sync                          # Creates venv and installs deps from pyproject.toml
source .venv/bin/activate

# Run development server
python main.py
# or: uvicorn main:app --reload --port 8000

# Testing
pytest                                    # Run all tests
pytest tests/test_agent_manager.py -v     # Run specific test file
pytest tests/test_agent_manager.py::test_function_name -v  # Run single test
```

### Development Ports

| Service | Port | Notes |
|---------|------|-------|
| Vite (desktop dev) | 1420 | HMR on 1421 |
| Python backend | 8000 | Dynamic in production |
| Vite (web dev) | 5173 | If running web version |

## Architecture Overview

### High-Level Data Flow

```
User Input → React Frontend → FastAPI Backend → AgentManager
                                                     ↓
                                            ClaudeSDKClient
                                                     ↓
                                            Claude Code CLI
                                                     ↓
                                            SSE Streaming → UI
```

### Desktop-Specific Architecture

```
Tauri App
├── React Frontend (Vite bundle)
├── Rust Core (lib.rs)
│   ├── Sidecar lifecycle management
│   ├── Dynamic port assignment
│   └── IPC bridge (Tauri commands)
└── Python Backend (PyInstaller sidecar)
    ├── FastAPI server
    ├── SQLite database
    └── ClaudeSDKClient
```

**Key Desktop Concepts:**
- Python backend runs as a **sidecar process** managed by Tauri
- Port is dynamically assigned via `portpicker` in Rust
- Frontend uses `getBackendPort()` from `services/tauri.ts` to get the port
- Data stored in platform-specific directories:
  - macOS: `~/Library/Application Support/Owork/`
  - Windows: `%LOCALAPPDATA%\Owork\` (typically `C:\Users\YourUsername\AppData\Local\Owork\`)
  - Linux: `~/.local/share/owork/`

### Backend Structure

```
backend/
├── main.py                   # FastAPI entry point
├── config.py                 # Settings
├── routers/                  # API endpoints (agents, skills, mcp, chat, plugins)
├── core/
│   ├── agent_manager.py     # ClaudeSDKClient wrapper, hooks, security
│   ├── session_manager.py   # Conversation session storage
│   └── workspace_manager.py # Per-agent isolated workspaces
├── database/
│   └── sqlite.py            # SQLite implementation
└── schemas/                  # Pydantic models
```

### Frontend Structure (Desktop)

```
desktop/
├── src/
│   ├── services/
│   │   ├── api.ts           # Axios client with dynamic port
│   │   ├── tauri.ts         # Tauri IPC bridge
│   │   └── *.ts             # Service modules with toCamelCase()
│   ├── pages/               # Route components
│   ├── components/          # UI components
│   └── types/               # TypeScript interfaces
├── src-tauri/
│   ├── src/lib.rs          # Rust: sidecar management, CLI detection
│   ├── binaries/           # PyInstaller output goes here
│   └── tauri.conf.json     # Tauri configuration
└── scripts/
    ├── build.sh            # Full build script
    └── build-backend.sh    # PyInstaller packaging
```

## API Data Naming Convention (CRITICAL)

**Backend uses `snake_case`, Frontend uses `camelCase`.**

Transformation functions in `desktop/src/services/*.ts` handle conversion:

| Service | File | Functions |
|---------|------|-----------|
| Agents | `agents.ts` | `toSnakeCase()`, `toCamelCase()` |
| Skills | `skills.ts` | `toCamelCase()` |
| MCP | `mcp.ts` | `toCamelCase()` |
| Chat | `chat.ts` | `toSessionCamelCase()`, `toMessageCamelCase()` |

**When adding new fields:**

1. Add to backend Pydantic model (`backend/schemas/*.py`) - `snake_case`
2. Add to frontend TypeScript interface (`desktop/src/types/index.ts`) - `camelCase`
3. **Update the corresponding `toCamelCase()` function** - this is commonly forgotten!

## Claude Agent SDK Usage

```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xiehust/owork](https://github.com/xiehust/owork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
