---
trigger: always_on
description: OpenBotX is an AI agent orchestration platform. Python backend (FastAPI), Vue 3 frontend (PrimeVue), real-time WebSocket communication.
---

# AGENTS.md

OpenBotX is an AI agent orchestration platform. Python backend (FastAPI), Vue 3 frontend (PrimeVue), real-time WebSocket communication.

## Build and Run

```bash
# Setup (first time)
make setup
source .venv/bin/activate

# Run backend (dev mode with reload)
make dev

# Run frontend (dev mode, separate terminal)
make web-client-dev

# Build frontend for production (output goes to openbotx/web_client/)
make web-client-build

# Lint and format
make lint
make format
```

## Code Standards

### General Rules

- Follow the existing patterns, architecture, layout, and visual style already in the project. Consistency is paramount.
- Write professional, clean code using industry best practices. No workarounds, no fallbacks, no hacky solutions, no legacy code.
- No unexpected behavior from catch-all `else` branches for unknown cases. Handle only known cases explicitly.
- Do not create documentation files (`.md`), migrations, scripts, or tests unless explicitly asked.
- Do not run commands — edit files directly.
- Code and comments must be in English.

### Python

- Ruff linter, line length 100, target Python 3.11. Rules: E, F, I, N, W, UP (ignore E501).
- `__init__.py` files must be completely empty — no code, no imports, nothing.
- Do not use `TYPE_CHECKING` / `if TYPE_CHECKING` import pattern.
- Only add comments where truly essential (to save tokens). Single-line `#` comments must be lowercase. Block/docstring comments use normal casing.
- Validate imports with: `python -c "from openbotx.server.app import create_app"`.

### Frontend

- Vue 3 Composition API (`<script setup>`), Pinia stores as composable functions (`defineStore` with `setup()` syntax).
- Follow the existing PrimeVue component usage, CSS variable patterns, and layout conventions.

## Architecture Overview

The system follows a **message bus pattern** where all components communicate through async queues, not directly.

```
Channels (Web/Telegram) → MessageBus (inbound) → Orchestrator → AgentLoop → LLM Provider
                                                                    ↓
                                                              Tool execution
                                                                    ↓
                        MessageBus (outbound) ← AgentLoop ← Tool results / final response
                              ↓
                        ChannelManager → Channels → User
```

### Key Relationships

**Message flow:** Every user message becomes an `InboundMessage` → goes through `MessageBus.inbound` → `Orchestrator` picks it up → routes to the right `AgentLoop` (via `AgentClassifier` if multi-agent) → `AgentLoop` calls LLM + tools in a loop → publishes `OutboundMessage` to `MessageBus.outbound` → `ChannelManager` routes it back to the originating channel.

**One AgentLoop per agent:** Each agent defined in config gets its own `AgentLoop` instance with its own `LLMProvider`, `ToolRegistry`, `PathResolver`, workspace directory, and model. They share the same `MessageBus`, `SessionManager`, and `TaskManager`.

**Sessions tie to channels, not agents:** Session key = `{channel}:{chat_id}`. When multiple agents handle the same chat, they share the session history. The `AgentClassifier` reads this history (with `[Agent: name]` prefixes) to maintain conversation continuity.

**Tasks track everything:** Every inbound message creates a `Task` (TODO → DOING → DONE/ERROR). Tasks provide the real-time observability layer. The frontend task board reads tasks via REST and receives updates via WebSocket events.

**Tools are the agent's hands:** The LLM decides which tools to call. `ToolRegistry` validates parameters, executes, and returns string results. Each tool manages its own output size limits. Failed tools get a recovery hint appended.

**Subagents are fire-and-forget:** `SpawnTool` creates an `asyncio.Task` that runs independently. When done, the subagent publishes its result back to the inbound queue as a new message. The main agent picks it up in a future turn and connects it to the original conversation via session history.

**EventDispatcher decouples broadcasting:** The agent loop broadcasts events (`chat:thinking`, `chat:tool_use`, `task:created`, etc.) through `EventDispatcher`, which routes to `WebSocketManager`. Components never call WebSocket directly.

**live_state is transient runtime data:** Both `Session` and `Task` have a `live_state` dict populated during execution (tool_uses, agent_name). It's returned via API but never saved to JSONL. Cleared when processing completes.

## Package Structure

```
openbotx/
├── agent/                  # AI agent orchestration
│   ├── orchestrator.py     # Consumes inbound bus, routes to agents
│   ├── classifier.py       # LLM-based agent selection (multi-agent only)
│   ├── loop.py             # Main agentic loop: LLM → tools → repeat
│   ├── context.py          # System prompt assembly (SOUL.md, USER.md, memory, skills)
│   ├── memory.py           # MEMORY.md / HISTORY.md read/write, consolidation prompts
│   ├── skills.py           # SKILL.md discovery from builtin + workspace dirs
│   └── subagent.py         # Background agent spawning with restricted tools
│

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openbotx/openbotx](https://github.com/openbotx/openbotx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
