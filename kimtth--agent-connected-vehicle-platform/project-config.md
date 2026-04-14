---
trigger: always_on
description: These instructions orient AI coding agents working in this repo. Focus on concrete, existing patterns—avoid inventing new architectures unless requested.
---

# Agentic Connected Vehicle Platform – AI Assistant Project Instructions

These instructions orient AI coding agents working in this repo. Focus on concrete, existing patterns—avoid inventing new architectures unless requested.

# Naming rules

## 🖥️ UI Components (camelCase)

- The interaction between UI and backend is handled via REST APIs, which use **camelCase** for JSON fields. The conversion between camelCase and snake_case is managed by Pydantic models in the backend.
- All UI elements follow the **camelCase** naming convention. 
- The `cosmos_db` component in the backend uses **camelCase** naming convention. 
- Backend components also follow **snake_case**. 

# 📘 Project Instruction UI

Welcome to the project! This README provides instructions for setting up and using the UI, `cosmos_db`, and backend components.


## 1. Architecture Snapshot
Backend (Python / FastAPI) lives in `vehicle/` and is the system of record + agent hub. Key layers:
- Entry point: `vehicle/main.py` – config, middleware, router loading, SSE streaming, MCP sidecar process launch (weather/traffic/poi/navigation on ports 8001–8004) and graceful shutdown.
- Routers: `vehicle/apis/*.py` grouped by domain (agents, vehicle features, remote access, emergency/safety, speech, dev seed). Loaded dynamically in `lifespan()`; missing modules are tolerated (log a warning, continue startup).
- Data access: `vehicle/azure/cosmos_db.py` – async singleton (`get_cosmos_client()`) with resilient connect / retry, container auto-provisioning, polling-based status subscription (no change feed listener yet), camelCase storage via Pydantic models.
- Auth: `vehicle/azure/azure_auth.py` – middleware validating Azure AD JWT if configured; supports optional mode. Accepts both raw GUID and `api://GUID` audiences, plus multiple acquisition channels (headers, query, cookies, DEV_BEARER_TOKEN env). Certain dev endpoints bypass auth.
- Agents: `vehicle/agents/*` – Microsoft Agent Framework `Agent` instances orchestrated by `AgentManager` (`agent_manager.py`). Manager aggregates domain agent tools (decorated with `@tool`) + general tools (`plugin/general_tools.py`) into a single coordinating agent and provides streaming + fallback logic.
- Plugins / AI client factory: `vehicle/plugin/oai_service.py` returns an `OpenAIChatClient` from `agent-framework-openai` which auto-detects Azure OpenAI (managed identity or API key) vs OpenAI from environment variables.
- Models: `vehicle/models/*.py` – All inherit from `CamelModel` which enforces outbound camelCase and accepts inbound snake_case or camelCase.
- Frontend (React) in `web/` (not deeply coupled—consumes REST + SSE `/api/vehicle/{id}/status/stream` and agent endpoints `/api/agent/*`).
- Dev seed + mock data: `vehicle/apis/dev_seed_routes.py` (creates demo vehicles/status) and MCP mock servers under `vehicle/plugin/mcp_*` (started as subprocesses when `ENABLE_MCP=true`).

## 2. Data & Serialization Conventions
- Always use model classes (`VehicleStatus`, `Command`, etc.) for API responses; they serialize camelCase automatically (`CamelModel`). Do NOT manually convert field names.
- Cosmos containers use partition key `/vehicleId`; queries frequently specify `partition_key=vehicleId` for efficiency—preserve this when adding queries.
- Status updates always create a new document (`update_vehicle_status` writes a fresh item with a generated `id` + timestamp). Do not mutate historical status docs.
- Commands/notifications sometimes have both `Id` and logical IDs (`command_id`); retain existing casing to avoid breaking stored items—prefer adding fields rather than renaming.

## 3. Async, Concurrency, and Streaming
- All Cosmos operations are async; ensure new endpoints `await client.ensure_connected()` before queries.
- SSE streaming of status: generator in `main.py` wraps `subscribe_to_vehicle_status()` (polls at 1s after updates, 5s idle). If extending, keep yields formatted as: `data: <json>\n\n`.
- Agent streaming: `agent_routes.ask_agent(stream=true)` uses `process_request_stream`; chunk model is `StreamingChunk` (camelCase). Maintain chunk shape for frontend compatibility.

## 4. Agent Orchestration Patterns
- `AgentManager.process_request()` builds a synthetic prompt: `Query: ...\nContext: {json}`. The single coordinating `Agent` has all domain tools registered; the LLM selects which tool to invoke via function calling.
- To add a new specialized agent: create a module under `vehicle/agents/` with `@tool`-decorated async functions (matching existing patterns), export a `*_TOOLS` list, import and concatenate the tools in `agent_manager.py._all_tools`. If a new public route form is exposed, update `agent_type_mapping` in `apis/agent_routes.py`.
- Fallback path (`_process_with_fallback`) creates a fresh `Agent` instance—keep this lightweight; avoid coupling to new heavy tools.

## 5. Azure Integration Nuances
- Auth optional by default; enforce by setting `AZURE_AUTH_REQUIRED=true`. When adding routes that must stay open for local dev, append to `exclude_prefixes` or `exclude_exact` cautiously.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kimtth) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
