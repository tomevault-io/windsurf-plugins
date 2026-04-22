---
trigger: always_on
description: **Tmux Agents** (`tmux-agents`) — a daemon-based AI agent orchestration platform with multiple client interfaces. Manages 10-50 concurrent AI agents (Claude, Gemini, Codex, OpenCode, Cursor, Copilot, Aider, Amp, Cline, Kiro) across local tmux, remote SSH servers, Docker containers, and Kubernetes pods.
---

# CLAUDE.md

## Project Overview

**Tmux Agents** (`tmux-agents`) — a daemon-based AI agent orchestration platform with multiple client interfaces. Manages 10-50 concurrent AI agents (Claude, Gemini, Codex, OpenCode, Cursor, Copilot, Aider, Amp, Cline, Kiro) across local tmux, remote SSH servers, Docker containers, and Kubernetes pods.

Built by super-agent.ai

---

## Architecture

### System Overview

```
┌──────────────────────────────────────────────────────────┐
│                    Client Layer                          │
│  ┌─────────────┐ ┌────────┐ ┌──────┐ ┌───────────────┐  │
│  │  VS Code    │ │  CLI   │ │ TUI  │ │  MCP Server   │  │
│  │  Extension  │ │        │ │      │ │ (Claude.app)  │  │
│  └──────┬──────┘ └───┬────┘ └───┬──┘ └───────┬───────┘  │
└─────────┼────────────┼──────────┼─────────────┼──────────┘
          │            │          │             │
    ┌─────┴────────────┴──────────┴─────────────┴───────┐
    │         Transport Layer (JSON-RPC 2.0)            │
    │  • Unix Socket (~/.tmux-agents/daemon.sock)       │
    │  • HTTP Fallback (localhost:3456)                 │
    │  • WebSocket Events (localhost:3457)              │
    └───────────────────────┬───────────────────────────┘
                            │
    ┌───────────────────────▼───────────────────────────┐
    │              Daemon Layer                         │
    │  ┌─────────────────────────────────────────────┐  │
    │  │  API Layer (JSON-RPC handler)               │  │
    │  └─────────────────┬───────────────────────────┘  │
    │  ┌─────────────────▼───────────────────────────┐  │
    │  │  Service Layer                              │  │
    │  │  • AgentOrchestrator (state machine)        │  │
    │  │  • PipelineEngine (DAG execution)           │  │
    │  │  • TaskRouter (priority routing)            │  │
    │  │  • TeamManager (team coordination)          │  │
    │  └─────────────────┬───────────────────────────┘  │
    │  ┌─────────────────▼───────────────────────────┐  │
    │  │  Persistence Layer (SQLite)                 │  │
    │  │  Database (~/.tmux-agents/tmux-agents.db)   │  │
    │  └─────────────────┬───────────────────────────┘  │
    └────────────────────┼───────────────────────────────┘
                         │
    ┌────────────────────▼───────────────────────────────┐
    │              Runtime Layer                         │
    │  ┌───────────┐ ┌──────────┐ ┌──────────────────┐  │
    │  │   Tmux    │ │  Docker  │ │   Kubernetes     │  │
    │  │ Runtime   │ │ Runtime  │ │    Runtime       │  │
    │  │ (local +  │ │          │ │                  │  │
    │  │   SSH)    │ │          │ │                  │  │
    │  └───────────┘ └──────────┘ └──────────────────┘  │
    └────────────────────────────────────────────────────┘
```

### Key Abstraction: All Runtimes Use Tmux

All runtimes (local, SSH, Docker, K8s) execute agents inside tmux sessions. The only difference is the **exec prefix**:

- **Local**: `tmux` (direct execution)
- **SSH**: `ssh user@host -- tmux` (remote execution)
- **Docker**: `docker exec <container-id> tmux` (containerized execution)
- **K8s**: `kubectl exec -n <namespace> <pod> -- tmux` (pod execution)

This unified abstraction allows the same `TmuxService` class to manage all agent types.

---

## Monorepo Structure

```
tmux-agents/                         # Monorepo root
├── src/                             # Main VS Code extension + shared core
│   ├── extension.ts                 # VS Code extension entry point
│   ├── core/                        # Core business logic (shared)
│   │   ├── types.ts                 # Shared TypeScript interfaces
│   │   ├── tmuxService.ts           # Tmux CLI wrapper
│   │   ├── database.ts              # SQLite persistence layer
│   │   ├── eventBus.ts              # Event emitter system
│   │   ├── config.ts                # Configuration management
│   │   ├── processTracker.ts       # Process categorization (50+ regex patterns)
│   │   └── index.ts                 # Core exports
│   ├── orchestrator.ts              # Agent state machine & dispatch
│   ├── pipelineEngine.ts            # Multi-stage DAG pipeline execution
│   ├── taskRouter.ts                # Priority-based task routing
│   ├── teamManager.ts               # Agent team composition
│   ├── agentTemplate.ts             # Agent template definitions
│   ├── aiAssistant.ts               # AI provider management (detection, launch)
│   ├── aiModels.ts                  # Centralized model registry
│   ├── apiCatalog.ts                # 100+ action catalog for AI responses
│   ├── tmuxContextProvider.ts       # Context gathering for AI prompts
│   ├── promptBuilder.ts             # Prompt construction
│   ├── promptRegistry.ts            # Template registry
│   ├── promptExecutor.ts            # Template execution engine
│   ├── treeProvider.ts              # VS Code tree view provider
│   ├── chatView.ts                  # AI Chat webview (streaming, tool loop)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/super-agent-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
