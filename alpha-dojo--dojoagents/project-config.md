---
trigger: always_on
description: - `git` commands are FORBIDDEN in this repository. Do NOT run `git status`, `git diff`, `git show`, `git checkout`, `git reset`, `git commit`, or any other `git` subcommand.
---

# Repository Agent Instructions

## Non-Negotiable Local Rules

- `git` commands are FORBIDDEN in this repository. Do NOT run `git status`, `git diff`, `git show`, `git checkout`, `git reset`, `git commit`, or any other `git` subcommand.
- Temporary scripts MUST be placed under `.agents/scripts/`.
- New third-party dependencies MUST NOT be introduced unless the primary lockfile is updated in the same change:
  - Python: `pyproject.toml` and `uv.lock`
  - Frontend: `dojoagents/dashboard/web/package.json` and `dojoagents/dashboard/web/package-lock.json`

## 1. High-Level Mental Map

### Core Purpose

DojoAgents is a quantitative finance agent runtime. It wires an LLM-driven agent loop, sandboxed tools, skills, memory, scheduled jobs, chat gateways, plugins, and a FastAPI/React dashboard for financial analysis workflows.

### Primary Tech Stack

- Python package: `dojoagents` version `0.0.1`
- Python runtime: `>=3.11`
- Backend/API: FastAPI `>=0.110.0,<0.112`, uvicorn `>=0.31.1,<0.33`
- LLM/API clients: OpenAI SDK `>=1.20.0,<2`, httpx `>=0.27.0,<1`
- Config/storage/data: PyYAML `>=6.0.1,<7`, pandas `>=2.2.0,<3`, pyarrow `>=14.0.0`, portalocker, APScheduler `>=3.10.0,<4`
- Agent/tooling: `mcp>=1.26.0,<2`, `strands-agents`, `strands-agents-tools`, `dojosdk`
- Frontend: React `^19.2.6`, React DOM `^19.2.6`, TypeScript `~5.8.3`, Vite `^8.0.12`
- Test stack: pytest `>=8.4.2`, pytest-asyncio `>=1.2.0`

## 2. Repository Directory Structure (Global Overview)

```text
.
├── AGENTS.md                         # This file: mandatory guidance for future agents.
├── README.md                         # User-facing setup, CLI, dashboard, and gateway overview.
├── VERSION                           # Package version marker.
├── pyproject.toml                    # Python package metadata, dependencies, console script.
├── requirements.txt                  # Runtime dependency mirror.
├── uv.lock                           # Primary Python lockfile.
├── docker/
│   └── Dockerfile                    # Container packaging.
├── docs/                             # Architecture, dashboard, protocol, plugin, and planning docs.
├── tests/                            # Pytest suite; mirrors agent, dashboard, gateway, plugin, tool surfaces.
├── .agents/
│   └── scripts/                      # REQUIRED location for temporary scripts.
└── dojoagents/
    ├── __init__.py                   # Package root.
    ├── logging.py                    # Unified logger configuration and LOGGER singleton.
    ├── agent/                        # Agent loop, runtime composition, providers, events, guardrails, harnesses.
    │   ├── runtime.py                # Main object graph wiring from ConfigStore.
    │   ├── loop.py                   # AgentLoop orchestration and streaming/tool flow.
    │   └── models.py                 # ChatRequest, ToolCall, ToolResult, LLMResult contracts.
    ├── cli/                          # `dojoagents` console entry point and interactive config/gateway setup.
    ├── config/                       # Central typed configuration system.
    │   ├── loader.py                 # ConfigStore, env expansion, deep merge, redaction, save.
    │   └── models.py                 # Frozen dataclass config schema.
    ├── cron/                         # Job storage and scheduler integration.
    ├── dashboard/                    # FastAPI server, routers, services, schemas, SSE, static assets, React app.
    │   ├── server.py                 # Dashboard app factory, lifespan, config/chat/jobs endpoints.
    │   ├── deps.py                   # FastAPI dependency accessors for initialized services.
    │   ├── routers/                  # API route modules. Add new dashboard routes here.
    │   ├── schemas/                  # Pydantic response/request models for dashboard APIs.
    │   ├── services/                 # Financial/domain services, stores, gateway wrappers, cache logic.
    │   ├── tools/                    # Dashboard-specific agent tool adapters.
    │   ├── static/                   # Packaged static HTML/assets.
    │   └── web/                      # React/Vite frontend source and frontend lockfile.
    ├── dojo_extensions/              # DojoExtension protocol and registry for first-class extensions.
    ├── gateway/                      # Chat gateway server, runner, state, pairing, platform adapters.
    │   └── adapters/                 # Slack/Telegram/WeChat/Feishu/etc adapters; subclass BaseGatewayAdapter.
    ├── memory/                       # Memory provider protocol, manager, local/skill-summary providers.
    ├── multi_agent/                  # Agent pool, delegation tools, orchestrator, triggers, automation.
    ├── planning/                     # Plan models, state store, execution engine, tools, triggers.
    ├── plugins/                      # Plugin discovery, manifests, hooks, plugin tool registration.
    │   └── built_in/                 # Built-in plugin examples and guardrails.
    ├── quant/                        # Quant context, risk, workflow primitives.
    ├── skills/                       # Skill manager/cache/loader and built-in procedural skills.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Alpha-Dojo/DojoAgents](https://github.com/Alpha-Dojo/DojoAgents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
