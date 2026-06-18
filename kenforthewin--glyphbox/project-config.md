---
trigger: always_on
description: Self-programming NetHack agent that uses an LLM to write and execute Python code to play NetHack. The LLM uses the `execute_code` tool call to interact with the game through a high-level Python API. The full game screen is automatically provided before each turn.
---

# CLAUDE.md

## Project Overview

Self-programming NetHack agent that uses an LLM to write and execute Python code to play NetHack. The LLM uses the `execute_code` tool call to interact with the game through a high-level Python API. The full game screen is automatically provided before each turn.

Two interfaces for observing the agent:
- **TUI** (`watch`): Terminal UI built with Textual for real-time monitoring
- **Web** (`serve`): FastAPI backend + React frontend for live streaming and historical run replay

## Commands

```bash
# Install dependencies
uv sync

# Run agent in TUI watch mode
uv run python -m src.cli watch

# Record TUI session with asciinema
uv run python -m src.cli watch --record

# Use a different model (cheaper for testing)
uv run python -m src.cli watch --model anthropic/claude-3-haiku-20240307

# Start web server with live agent
uv run python -m src.cli serve
uv run python -m src.cli serve --port 8000 --model google/gemini-3-flash-preview

# Development mode: backend + frontend dev servers with hot reload
./dev.sh  # Starts FastAPI on :8000, Vite on :3000

# Build frontend for production (served by FastAPI from frontend/dist/)
cd frontend && npm install && npx vite build

# Verify setup
uv run python -m src.cli verify

# Run tests (skips integration tests by default)
uv run pytest

# Run specific test file
uv run pytest tests/test_agent_agent.py -v

# Run integration tests (requires API key)
uv run pytest -m integration

# Lint
uv run ruff check src/ tests/
uv run ruff format src/ tests/
```

## Architecture

```
LLM (strategic layer)
    | receives: game screen, inventory, monsters, adjacent tiles each turn
    | tool calls: execute_code, view_full_map, write_skill, invoke_skill
    v
NetHackAgent (src/agent/agent.py)
    | orchestrates: step loop, context compression, error recovery
    v
SkillSandbox (src/sandbox/manager.py)
    | validates & executes Python code (AST checks, forbidden imports)
    v
NetHackAPI (src/api/nethack_api.py)          Persistence (src/persistence/)
    | high-level game interface                 | SQLite: runs + turns
    v                                           v
NLE (NetHack Learning Environment)           Web API (src/web/)
                                               | FastAPI REST + WebSocket
                                               v
                                             Frontend (frontend/)
                                               | React + TypeScript + Vite
```

### Core Components

- **`src/cli.py`**: Entry point. Commands: `watch` (TUI mode), `serve` (web server), `verify`
- **`src/agent/agent.py`**: `NetHackAgent` class - main orchestration loop. Calls `step()` repeatedly which gets LLM decision and executes it. Manages conversation history with context compression
- **`src/agent/llm_client.py`**: `LLMClient` using OpenAI-compatible API. Supports OpenRouter and direct Anthropic providers. Defines tools and handles extended thinking/reasoning
- **`src/agent/prompts.py`**: `PromptManager` - builds system prompts, decision prompts, and formats last_result feedback. Handles deduplication of repeated game messages
- **`src/agent/parser.py`**: Parses LLM tool call responses, extracts JSON from markdown code blocks
- **`src/sandbox/manager.py`**: `SkillSandbox.execute_code()` runs agent-generated Python in restricted namespace with `nh` (NetHackAPI) available. `APICallTracker` wraps the API to record all action calls
- **`src/api/nethack_api.py`**: High-level NetHack interface wrapping NLE observations/actions
- **`src/memory/`**: Episode memory system - `EpisodeMemory` coordinates `WorkingMemory` (current state), `DungeonMemory` (per-level exploration), and `MemoryManager` (SQLite persistence)
- **`src/persistence/`**: Turn-level persistence layer - `TurnRepository` protocol with `SQLiteTurnRepository` implementation. Stores `RunRecord` and `TurnRecord` in `data/turns.db`
- **`src/scoring/progress.py`**: BALROG progression scoring - estimates win probability based on dungeon depth and XP level using historical human gameplay data
- **`src/tui/app.py`**: Textual TUI for watching agent play. Keybindings: S=start, Space=pause, Q=quit
- **`src/web/`**: FastAPI web server with REST API and WebSocket live streaming
- **`frontend/`**: React TypeScript SPA with Vite build tooling

### Data Flow

1. `NetHackAgent.step()` builds game state context (screen, position, inventory, monsters, adjacent tiles, stairs, reminders/notes)
2. `PromptManager` formats the decision prompt and compresses conversation history
3. LLM receives context and returns tool call (`execute_code` with Python code)
4. `SkillSandbox.execute_code()` validates code (AST checks, forbidden imports) and runs it
5. Code has access to `nh` (NetHackAPI), `Direction`, `Position`, and other pre-loaded types - all calls are synchronous
6. `APICallTracker` records every action call with success/failure status
7. Results (game messages, API call log, autoexplore results) returned to agent for next LLM context

**Web path (when using `serve`):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kenforthewin/glyphbox](https://github.com/kenforthewin/glyphbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
