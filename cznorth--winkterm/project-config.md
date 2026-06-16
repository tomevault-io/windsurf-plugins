---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WinkTerm is an AI + terminal "human-machine fusion" ops tool. The AI and the user share the same pty session, and all interaction happens inside the terminal. The user talks to the AI by typing a `# message`; the AI can suggest commands and write them into the input line, where the user presses Enter to run or Backspace to edit.

## Common Commands

### Backend Development
```bash
cd backend
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# Start the development server
python -m uvicorn backend.main:app --reload --port 8000
```

### Frontend Development
```bash
cd frontend
npm install
npm run dev           # Development mode
npm run build         # Build (output to frontend/out/)
npm run lint          # Lint check
npm run gen:api       # Generate TypeScript types and react-query hooks from OpenAPI
```

### Desktop App Packaging
```bash
# Windows
build\build.bat

# Or run manually
pyinstaller build\winkterm.spec --clean --noconfirm
```

### Docker Deployment
```bash
docker compose up -d
```

## Architecture

### Core Data Flow
```
User keyboard input
    │
    ▼
Frontend Terminal (xterm.js)
    │  WebSocket
    ▼
ws_handler.py
    │
    ├── Normal input ──► pty_manager.write() ──► shell process
    │
    └── Lines starting with # ──► intercept ──► Agent (LangGraph)
```

### Key Modules

| Module | Path | Responsibility |
|--------|------|----------------|
| WebSocket handling | `backend/terminal/ws_handler.py` | Message dispatch, `#` detection, Agent invocation |
| PTY management | `backend/terminal/pty_manager.py` | Shell process wrapper, read/write, context retrieval |
| Session management | `backend/terminal/session_manager.py` | Multi-terminal sessions, active state |
| Agent graph | `backend/agent/graph.py` | LangGraph StateGraph definition |
| Agent tools | `backend/agent/tools/` | Terminal interaction tool definitions |
| SSH connections | `backend/ssh/` | SSH connection management, file transfer |

### Agent Tools

- `terminal_input`: Run a command or send control keys, returns the execution result
- `write_command`: Write a command into the input line (without running it); the agent stops and waits for the user
- `get_terminal_context`: Read terminal output content (read-only)

### WebSocket Message Protocol

| Direction | type | Meaning |
|-----------|------|---------|
| Frontend → Backend | input | User keyboard input |
| Frontend → Backend | resize | Terminal size change |
| Backend → Frontend | output | Raw pty output |

AI messages are returned via pty output, preserving the human-machine fusion experience.

### Frontend Structure

- `src/app/`: Next.js App Router
- `src/components/Terminal/`: xterm.js wrapper
- `src/lib/websocket.ts`: WebSocket client (with reconnect)
- `src/lib/api/generated.ts`: orval-generated API hooks

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `ANTHROPIC_API_KEY` | Anthropic API Key | Yes |
| `MODEL_NAME` | Claude model name | No (default `claude-opus-4-6`) |
| `NEXT_PUBLIC_API_URL` | Backend HTTP API address | No |
| `NEXT_PUBLIC_WS_URL` | Backend WebSocket address | No |

## Frontend Debugging (Agent Self-Verification)

When verifying frontend fixes, **pick one method based on the runtime environment** (don't mix them):

| Environment | Method |
|-------------|--------|
| **Cursor IDE** | Built-in browser (Browser MCP) |
| **Others** (Claude Code, CI, agents without MCP) | puppeteer-core + system Chrome (see below) |

### Cursor: Built-in Browser (preferred)

When testing `http://localhost:3000` in Cursor, have the agent use **only the built-in browser** — don't start puppeteer.

**Prerequisites**: backend and frontend are running; `frontend/.env.example` has been copied to `frontend/.env.local` (pointing at `localhost:8000`).

**Recommended flow**:

1. `browser_navigate` → `http://localhost:3000`
2. `browser_lock` → interact → `browser_unlock`
3. Standard controls: `browser_snapshot` → `browser_click` / `browser_type` / `browser_fill`
4. **xterm terminal**: first click `.xterm-screen` (or click via `browser_cdp`), then type with `browser_press_key`; read output via `browser_cdp` + `Runtime.evaluate` on `.xterm-rows` (don't use whole-page `textContent`)
5. **Activity bar / split layout**: snapshots often lack a ref — use `browser_cdp` to click `.activity-item`, `.layout-btn`, etc.

**Smoke items to cover**: auth, local terminal echo, new tab, `+` dropdown, SSH list, settings page, AI sidebar and chat, split layout.

**Password/secret retention**: when editing SSH or settings without changing the password/API Key fields, they must not be cleared after save (the frontend and backend already implement retention logic; you can additionally verify `~/.winkterm/config.json` via the API).

Backend interaction, incremental terminal reads, etc. can still use the Agent HTTP API under **"Running Scenarios"** below; it complements browser-based UI testing.

### Other Environments: puppeteer-core + system Chrome


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cznorth/winkterm](https://github.com/Cznorth/winkterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
