---
trigger: always_on
description: > This file provides guidance to AI coding agents working with this repository.
---

# AGENTS.md

> This file provides guidance to AI coding agents working with this repository.
> Expect the reader to know nothing about the project.

## Project Overview

**MiniMax Agent GUI** is a personal AI agent application powered by the MiniMax M3 model (with M2.7 and M2.7-highspeed as selectable options). It provides:
1. A **desktop app** (`desktop/`, Tauri 2 + Vite + React 18 + Tailwind) — primary and only installable interface
2. A **CLI framework** (`mini_agent/cli.py`) — terminal-based interactive agent

The project wraps MiniMax MCP tools (`web_search`, `understand_image`) as standard agent tools and provides media generation (TTS, Image, Music, Video) via MiniMax APIs. The FastAPI backend (`web/backend/`) is bundled by the Tauri shell as a sidecar — there is no separate web app in this repo.

- **Name**: `minimax-agent-gui`
- **Version**: `0.4.0`
- **License**: MIT
- **Python**: `>=3.10`
- **Node**: `>=18`
- **Status**: Active development — desktop-first migration complete (Tauri scaffold + speech 4 sub-modes + settings index rail + skills system + agent context system)

## Technology Stack

### Desktop App
- **Shell**: Tauri 2 (Rust) + Vite + React 18 + Tailwind CSS
- **Component library**: shadcn-style components (`lucide-react`, `radix-ui`, `class-variance-authority`)
- **Markdown**: `react-markdown` + `remark-gfm`
- **i18n**: `react-i18next` (6 languages)
- **Icons**: `lucide-react`
- **Bundled backend**: PyInstaller-bundled FastAPI sidecar on `:8765`

### Backend (`web/backend/`)
- **Framework**: FastAPI, Uvicorn, WebSocket
- **HTTP Client**: `httpx` (sync and async)
- **Data Validation**: Pydantic v2
- **Configuration**: YAML (`pyyaml`)

### Core Framework
- **LLM Providers**: Anthropic (Claude) and OpenAI-compatible APIs via `mini_agent.llm`
- **API Backend**: MiniMax API (`api.minimax.io` / `api.minimaxi.com`)
- **Token Counting**: `tiktoken` (cl100k_base encoder)
- **CLI Framework**: `prompt-toolkit`
- **Build System**: `hatchling`

## Project Structure

```
minimax-agent-gui/
├── desktop/                    # Tauri 2 desktop app (only installable interface)
│   ├── src/                    # React 18 + Vite + Tailwind frontend
│   │   ├── components/         # Chat, Coding, media, settings, agent-context, shared
│   │   ├── hooks/              # useSessionProtection, AgentActivityContext, useAgentContext
│   │   ├── i18n/               # 6-language i18n config
│   │   ├── App.jsx             # Top-level shell + tab routing
│   │   ├── themes.css          # 9 color themes
│   │   └── main.jsx
│   ├── src-tauri/              # Rust backend (tauri 2.1 + tauri-plugin-shell 2.0)
│   │   └── src/lib.rs          # start_backend() spawns the FastAPI sidecar
│   ├── vite.config.js          # Vite + proxy (/api, /ws → :8765)
│   ├── tauri.conf.json         # productName "MiniMax Agent", identifier com.minimax.agent.desktop
│   └── package.json
├── web/                        # Backend only (bundled by Tauri sidecar)
│   └── backend/
│       ├── main.py             # FastAPI: REST API + WebSocket chat
│       ├── agent_context.py    # .agent/*.md loader (SOUL/IDENTITY/USER/MEMORY)
│       ├── conv_store.py       # Conversation persistence (JSON)
│       ├── i18n.py             # Backend i18n (89 keys × en-US/pt-BR)
│       ├── mcp_agent_tools.py  # MCP runtime
│       ├── mcp_runtime.py      # MCP server lifecycle
│       └── subdirectory_hints.py  # Progressive subdirectory discovery (AGENTS.md, CLAUDE.md)
├── mini_agent/                 # Core agent framework (reusable library)
│   ├── agent.py                # Async agent loop, tool execution, token summarization
│   ├── cli.py                  # Interactive CLI entry point
│   ├── config.py               # Pydantic-based config loader
│   ├── llm/                    # LLMClient (Anthropic/OpenAI routing)
│   └── tools/                  # Bash, File, Note, MCP, Skill tools
├── mini_max_mcp/               # MiniMax-specific integrations
│   ├── client.py               # MiniMaxSyncClient / MiniMaxClient
│   │                           # TTS, Image (T2I + I2I), Video, Music APIs
│   ├── mcp_tools.py            # MiniMaxMCPClient (web_search, understand_image)
│   └── mcp_tool_wrapper.py     # Tool wrappers for Agent
├── tests/                      # pytest test suite
├── config/                     # User configuration (gitignored, contains secrets)
│   └── config.yaml
├── workspace/                  # Runtime working directory
│   ├── conversations/          # Auto-saved chat histories (JSON)
│   ├── uploads/                # Uploaded files from chat/code panels
│   └── logs/                   # Application logs
└── examples/                   # Progressive usage examples
```

## Entry Points

### Desktop App
```bash
cd desktop
npm run tauri:dev          # Launches Tauri shell + auto-spawns backend sidecar
```

### CLI
```bash
mini-agent
```

## Build and Install Commands

```bash
# Python dependencies
pip install -e .

# Desktop app dependencies
cd desktop && npm install

# Run desktop app (Tauri dev mode)
cd desktop && npm run tauri:dev

# Run tests
pytest -v
```

## Configuration

### User Config (`config/config.yaml`)

Gitignored file containing secrets:

```yaml
minimax:
  api_key: sk-cp-...

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eduardoabreu81/minimax-agent-gui](https://github.com/eduardoabreu81/minimax-agent-gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
