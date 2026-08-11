---
trigger: always_on
description: **cognis** is a controller and orchestration layer for self-hosted AI agents — a decoupled control plane that manages agent definitions, interactive chat, delegated sub-sessions, tool execution routing, and integrations with external memory (Mnemory) and guardrails/audit (Intaris) services.
---

# AGENTS.md — Coding Agent Instructions for cognis

## Project Overview

**cognis** is a controller and orchestration layer for self-hosted AI agents — a decoupled control plane that manages agent definitions, interactive chat, delegated sub-sessions, tool execution routing, and integrations with external memory (Mnemory) and guardrails/audit (Intaris) services.

- **Language**: Python 3.12+, typed, async-first
- **Framework**: FastAPI (Starlette) for HTTP/WebSocket, Typer for CLI
- **Core dependencies**: fastapi, uvicorn, httpx, pydantic v2, sqlalchemy 2.x, litellm, typer
- **Frontend**: SvelteKit (separate app in `ui/`)
- **License**: TBD
- **Repository**: https://github.com/fpytloun/cognis
- **Companion services**: Intaris guardrails/audit and Mnemory memory

## Architecture

```
cognis/
├── pyproject.toml
├── cognis/
│   ├── __init__.py
│   ├── main.py                     # Entry point (Typer CLI + server start)
│   ├── config.py                   # Env var configuration (no config file)
│   │
│   ├── api/                        # API Gateway
│   │   ├── app.py                  # FastAPI factory
│   │   ├── routes/
│   │   │   ├── auth.py             # Login, refresh, logout, setup, exchange-token
│   │   │   ├── conversations.py
│   │   │   ├── agents.py
│   │   │   ├── secrets.py
│   │   │   ├── settings.py         # System settings, LLM providers, model routing
│   │   │   ├── tasks.py            # Task queue, dependencies, gate/step responses
│   │   │   ├── tools.py
│   │   │   ├── workflows.py        # Workflow CRUD, duplication, import/export
│   │   │   ├── schedules.py        # Schedule CRUD (task factory)
│   │   │   ├── escalations.py
│   │   │   ├── images.py            # Image serving, upload, generation
│   │   │   └── system.py           # Health, metrics, JWKS
│   │   ├── websocket.py            # WebSocket transport layer (thin adapter)
│   │   ├── executor_ws.py          # Executor WebSocket endpoint (auth + configure)
│   │   ├── runtime_support.py      # Step runtime factory (executor resolution)
│   │   ├── middleware.py           # Auth (JWT + API key), rate limiting
│   │   └── models.py              # API request/response Pydantic models
│   │
│   ├── core/                       # Orchestration Core
│   │   ├── turn_scheduler.py      # Turn orchestration (transport-agnostic)
│   │   ├── commands.py            # Slash command dispatch (transport-agnostic)
│   │   ├── agent_loop.py          # Agent loop engine (step runner)
│   │   ├── task_queue.py          # Queue picking, capacity, dependency resolution
│   │   ├── workflow_engine.py     # Workflow orchestration (step sequencing, gates, loops)
│   │   ├── step_evaluator.py      # Semantic step completion evaluation
│   │   ├── decision.py            # Decision Engine (rules + workflow selection)
│   │   ├── session.py             # Session Manager
│   │   ├── session_cache.py       # L1 in-memory cache for Intaris-derived state
│   │   ├── tool_router.py         # Tool routing logic
│   │   ├── compaction/            # Context compaction package (strategy, banding, fallback, recovery)
│   │   ├── context.py             # Context assembly (parallel external fetches)
│   │   ├── events.py              # Event Bus + hooks
│   │   ├── remember_queue.py      # Bounded retry queue for Mnemory remember
│   │   ├── agent_registry.py      # System agent definitions + registry
│   │   └── executor_resolution.py # Executor selection (labels, defaults, tool enablement)
│   │
│   ├── models/                     # Domain models (Pydantic)
│   │   ├── agent.py
│   │   ├── channel.py
│   │   ├── session.py
│   │   ├── tool.py
│   │   ├── delegation.py
│   │   └── config.py              # LLMProviderConfig, ModelRoutingPolicy, ImageGenerationResult, etc.
│   │
│   ├── artifacts/                  # Binary artifact storage (images, etc.)
│   │   ├── __init__.py
│   │   └── store.py               # ArtifactBackend Protocol, FS/S3 backends, ArtifactStore
│   │
│   ├── channels/                   # External messaging adapters + pairing
│   │   ├── protocol.py            # ChannelAdapter protocol + base adapter
│   │   ├── manager.py             # Channel lifecycle + adapter orchestration
│   │   ├── inbound.py             # Channel -> TurnScheduler pipeline
│   │   ├── delivery.py            # EventBus -> channel delivery
│   │   ├── pairing.py             # Sender-initiated remote verification flow
│   │   ├── formatting.py          # Message splitting + markdown stripping
│   │   ├── registry.py            # Channel metadata for setup UI
│   │   └── adapters/              # Signal/Slack/etc. concrete adapters
│   │
│   ├── providers/                  # Provider interfaces + implementations
│   │   ├── base.py                 # Protocol definitions (all 6 providers)
│   │   ├── retry.py               # Shared retry utility (exponential backoff + jitter)
│   │   ├── registry.py
│   │   ├── memory/
│   │   │   ├── protocol.py        # MemoryProvider Protocol

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fpytloun/cognis](https://github.com/fpytloun/cognis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
