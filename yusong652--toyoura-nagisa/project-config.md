---
trigger: always_on
description: This file provides guidance to Agent when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Agent when working with code in this repository.

## Project Overview

**toyoura-nagisa** is an AI agent platform for professional scientific computing, focusing on ITASCA PFC (Particle Flow Code) discrete element simulations with real-time WebSocket communication.

### Technology Stack

- **Backend**: Python 3.10+, FastAPI, uvicorn, ChromaDB
- **Frontend**: React 19, TypeScript, Material-UI, Vite
- **AI Infrastructure**: Multi-provider LLM support, real-time streaming, in-process tool orchestration (optional MCP gateway)
- **Scientific Computing**: WebSocket integration with ITASCA PFC Python SDK
- **Communication**: WebSocket for real-time updates, RESTful API for state management

## Architecture & Core Features

### Clean Architecture Pattern

```
Presentation Layer (API, WebSocket, Request Handlers)
    ↓ orchestrates through
Application Layer (Use Cases, Tooling, Agent Runtime)
    ↓ uses
Domain Layer (Models, Business Rules, Contracts)
    ↑ implemented by adapters in
Infrastructure Layer (LLM, MCP Gateway, Memory, OAuth, PFC, Storage, WebSocket)
```

**Key Principles**:
- **Dependency Inversion**: Infrastructure depends on domain abstractions
- **Single Responsibility**: Each layer has clear boundaries
- **Testability**: Domain logic isolated from external systems

**Layer Responsibilities**:
- **Presentation**: API routes, WebSocket handlers, request/response formatting
- **Application**: Business logic orchestration (ChatOrchestrator, content processing, tool execution)
- **Domain**: Core models and business rules (StreamingChunk, BaseMessage)
- **Infrastructure**: External integrations (LLM providers, optional MCP gateway, storage)

**Example**: Swapping LLM providers requires zero application/domain layer changes (`packages/backend/infrastructure/llm/base/client.py`)

### LLM Providers

Located in `packages/backend/infrastructure/llm/providers/`: google (primary), anthropic, moonshot, openai, openai_codex, openrouter, zhipu, google_antigravity, google_gemini_cli, web_search.

**Configuration**: `packages/backend/infrastructure/storage/llm_config_manager.py` (runtime provider/model selection)

### SubAgent System

MainAgent can delegate specialized tasks to lightweight SubAgents via the `invoke_agent` tool.

**Available SubAgents**:

| SubAgent | Tools | Max Iterations | Purpose |
|----------|-------|----------------|---------|
| **PFC Explorer** | 14 read-only | 64 | Documentation search, codebase exploration |
| **PFC Diagnostic** | 9 read-only | 64 | Multimodal visual analysis, task status inspection |

**PFC Explorer Tools**: `read`, `glob`, `grep`, `bash`, `bash_output`, `pfc_browse_commands`, `pfc_browse_python_api`, `pfc_query_python_api`, `pfc_query_command`, `pfc_browse_reference`, `pfc_list_tasks`, `pfc_check_task_status`, `web_search`, `todo_write`

**PFC Diagnostic Tools**: `pfc_capture_plot`, `read`, `pfc_check_task_status`, `pfc_list_tasks`, `glob`, `grep`, `bash`, `bash_output`, `todo_write`

**Architecture**:
- **Non-Streaming**: SubAgents use synchronous API calls for efficiency
- **Memory Isolation**: SubAgent todos stored in memory (`_memory_todos`), not persisted to disk
- **Context Separation**: SubAgent exploration doesn't consume MainAgent's context window
- **Read-Only**: SubAgents cannot execute simulations or modify files (no `pfc_execute_task`, `write`, `edit`)

**Storage Mode** (determined by agent type):
- MainAgent (`pfc_expert`): `persistent=True` → local file (`workspace/todos.json`)
- SubAgent (`pfc_explorer`, `pfc_diagnostic`): `persistent=False` → in-memory storage

**Implementation**:
- Tool: `packages/backend/application/tools/agent/invoke_agent.py`
- Config loader: `packages/backend/domain/models/agent_profiles.py` (source data in `config/agents.yaml`)
- Prompts: `packages/backend/config/prompts/pfc_explorer.md`, `packages/backend/config/prompts/pfc_diagnostic.md`

### Tool System

**In-process Tool Categories** (`packages/backend/application/tools/`):
- `coding/`: write, read, edit, bash, bash_output, kill_shell, glob, grep
- `pfc/`: pfc_execute_task, pfc_interrupt_task, pfc_check_task_status, pfc_list_tasks, pfc_capture_plot, pfc_query_*, pfc_browse_*
- `planning/`: todo_write
- `agent/`: invoke_agent, trigger_skill
- `builtin/`: web_search, web_fetch

**Optional MCP Gateway**:
- `packages/backend/infrastructure/mcp/` can expose internal tools to external MCP clients
- Internal tool execution does not require an MCP server

### PFC Integration Overview

**Problem**: AI-assisted control of professional scientific software (ITASCA PFC) requires solving:
1. Thread-safety (PFC SDK requires main thread execution)
2. Long-running tasks (simulations can run for hours)
3. Progress visibility (commands are non-interruptible)
4. Documentation-driven workflow (LLM needs command syntax guidance)

**Solution Architecture**:
```
Documentation Query → Test Script → Production Script → WebSocket → PFC SDK
     ↓                    ↓              ↓
Query Tools      Small-Scale Test   Full Simulation
(syntax ref)     (quick validate)   (task tracking)
```

**Key Components**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yusong652) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
