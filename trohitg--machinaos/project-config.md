---
trigger: always_on
description: handles ADB dispatch via `SERVICE_ID_MAP`. See the
---

# MachinaOs - Claude Documentation

## Project Overview
This is a React Flow-based workflow automation platform implementing n8n-inspired architectural patterns. The project has undergone a comprehensive refactoring to implement modern INodeProperties interface system with full TypeScript compliance and code cleanup.

## Documentation Reference

**Always refer to these documentation files for detailed guides:**

| Document | Description |
|----------|-------------|
| **[Frontend Architecture](./docs-internal/frontend_architecture.md)** | Current frontend stack (React 19 + Vite + Tailwind v4 + shadcn/ui + Radix + RHF/zod + TanStack Query + Zustand). Tokens, primitives, forms, credentials exemplar, ownership boundary, `uiHints` catalogue. |
| **[UI Migration Plan](./docs-internal/ui_migration_plan.md)** | antd → shadcn/ui migration plan + completion log. Waves 1–10 done. Full frontend is schema-driven (backend SSOT); remaining DIY widget registry (ex-Phase 6) is the one deferred item. |
| **[Schema Source of Truth RFC](./docs-internal/schema_source_of_truth_rfc.md)** | Backend is SSOT for node schemas, visual metadata, handlers, palette metadata, icons. Plugin pattern: one `BaseNode` subclass in `server/nodes/<group>/<name>.py`. Wire format: `asset:<key>` / `<lib>:<brand>` / URL / emoji. Endpoint: `/api/schemas/nodes/{type}/spec.json`. 124 pytest invariants lock the contract. |
| **[Plugin System (Wave 11)](./docs-internal/plugin_system.md)** | Class-based plugin-first architecture. `BaseNode` / `ActionNode` / `TriggerNode` / `ToolNode` + `@Operation` decorator. Pydantic `Params`/`Output`. Declarative `Routing` DSL + `Connection` facade (Nango pattern). 18 `Credential` subclasses live in each node folder's `_credentials.py` (or inline for single-use). `TaskQueue` constants route to Temporal worker pools. 111 plugins live across 9 queues; handler bodies fully inlined (`services/handlers/` shrank 12.8K → 1.1K LOC across 16 → 4 files; only cross-cutting orchestration remains: `tools.py` AI-tool dispatch + agent delegation, `google_auth.py`, `triggers.py`). **Wave 11.H added "self-contained plugin folders"** — five generic registries (`ws_handler_registry`, `event_waiter.{register_filter_builder,register_trigger_precheck}`, `status_broadcaster.register_service_refresh`, `node_output_schemas.register_output_schema`) so rich plugins like telegram own their entire surface area without core-services edits. |
| **[Nodes Cookbook](./server/nodes/README.md)** | 5-minute recipe + folder map + shared helpers (`_base.py` / `_inline.py` per domain) + shared credentials + **self-contained plugin folder pattern (telegram reference)** + contract invariants + common pitfalls. Lives next to the plugin files. |
| **[Node Creation Guide](./docs-internal/node_creation.md)** | Canonical plugin recipe — one Python file (default), or a self-contained folder for richer plugins (telegram-style). Zero frontend edits, zero core-services edits. Auto-registers via `BaseNode.__init_subclass__` + the five `register_*` hooks. |
| **[AI Tool Node Guide](./docs-internal/ai_tool_node_creation.md)** | Detailed guide for creating dedicated AI Agent tool nodes (schemas, handlers, toolkits) |
| **[Specialized Agent Guide](./docs-internal/specialized_agent_node_creation.md)** | Guide for creating specialized AI agents (Android, Coding, Web, Task, Social, Travel, Tool, Productivity, Payments, Consumer) with full AI configuration |
| **[Dual-Purpose Tool Guide](./docs-internal/dual_purpose_tool_node_creation.md)** | Guide for nodes that work as both workflow nodes AND AI Agent tools (e.g., whatsappSend) |
| **[Agent Architecture](./docs-internal/agent_architecture.md)** | How AI Agent and Chat Agent discover skills/tools, inject them into LLM prompts, and execute via LangGraph |
| **[Agent Delegation](./docs-internal/agent_delegation.md)** | How memory, parameters, and execution context flow when one AI agent delegates work to another agent connected as a tool |
| **[Agent Teams](./docs-internal/agent_teams.md)** | Claude SDK Agent Teams pattern - AI Employee and Orchestrator nodes with input-teammates handle for multi-agent coordination |
| **[Memory Compaction](./docs-internal/memory_compaction.md)** | Token tracking and model-aware memory compaction using native provider APIs (Anthropic, OpenAI) with threshold = 50% of context window |
| **[Pricing Service](./docs-internal/pricing_service.md)** | Centralized cost tracking for LLM tokens and API services (Twitter, Google Maps) with HTTPX event hooks |
| **[Proxy Service](./docs-internal/proxy_service.md)** | Residential proxy provider management with template-based URL formatting, health scoring, and transparent HTTP node injection |
| **[Email Service](./docs-internal/email_service.md)** | IMAP/SMTP integration via Himalaya CLI with EmailService orchestrator, provider presets, custom credential fallback, and polling triggers |
| **[CI/CD Pipeline](./docs-internal/ci_cd.md)** | GitHub Actions workflows, predeploy validation, release publishing, and composite setup action |
| **[Workflow Schema](./docs-internal/workflow-schema.md)** | JSON schema for workflows, edge handle conventions, config node architecture |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trohitg/MachinaOS](https://github.com/trohitg/MachinaOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
