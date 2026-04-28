---
trigger: always_on
description: Persistent AI entity harness daemon built on Deno 2.x. Web chat UI with streaming LLM, tool execution, hierarchical memory, RAG, knowledge graph, lorebook, data vault, and identity management. Runs on port 3000.
---

# Psycheros — Agent System Card

Persistent AI entity harness daemon built on Deno 2.x. Web chat UI with streaming LLM, tool execution, hierarchical memory, RAG, knowledge graph, lorebook, data vault, and identity management. Runs on port 3000.

Psycheros is an **embodiment** — an interface through which the AI entity exists and interacts. The entity's canonical identity lives in [entity-core](https://github.com/zarilewis/entity-core) (MCP server). Psycheros syncs with it when `PSYCHEROS_MCP_ENABLED=true`.

## First-Person Convention

All prompts, system messages, tool descriptions, and code comments use the entity's first-person perspective ("I am...", "I should..."), never second-person. The entity internalizes the system as *theirs*, not as rules imposed on them. See [docs/entity-philosophy.md](docs/entity-philosophy.md) for the full rationale. **Maintain this convention in all contributions.**

## Commands

```bash
deno task dev          # Development with hot reload
deno task start        # Production mode
deno task stop         # Graceful shutdown
deno check src/main.ts # Type check
deno lint              # Lint
```

## Setup

```bash
cp .env.example .env   # Then set LLM API key and PSYCHEROS_TOOLS
```

LLM connections are configured via **Settings > LLM Settings** in the web UI. Multiple named connection profiles can be created (OpenRouter, OpenAI, Alibaba/Qwen, NanoGPT, or custom endpoints). One profile is marked active for chat. On first run, a default profile is created from `ZAI_*` environment variables if set.

### With MCP (entity-core)

```bash
PSYCHEROS_MCP_ENABLED=true deno task dev
```

## Key Files

| File | Purpose |
|------|---------|
| `src/main.ts` | Entry point, MCP initialization |
| `src/types.ts` | Shared types (SSEEvent, LLMContextSnapshot, ToolCall) |
| `src/entity/loop.ts` | Agentic loop — LLM calls, tool execution, context capture, image/tool-arg fading |
| `src/entity/context.ts` | Context loading (supports MCP client) |
| `src/server/routes.ts` | API endpoints and handlers |
| `src/server/state-changes.ts` | Unified state mutations |
| `src/server/broadcaster.ts` | Persistent SSE for background updates |
| `src/tools/registry.ts` | Tool catalog (`AVAILABLE_TOOLS`) and registration |
| `src/tools/tools-settings.ts` | Tool enable/disable persistence and resolution |
| `src/tools/custom-loader.ts` | Dynamic loader for user-written tools in `custom-tools/` |
| `src/tools/web-search.ts` | Web search tool (Tavily / Brave) |
| `src/tools/send-discord-dm.ts` | Discord DM tool (sends DMs via Discord bot API) |
| `src/tools/control-device.ts` | Home automation tool (smart plug control via Shelly API) |
| `src/tools/generate-image.ts` | Image generation tool (OpenRouter, Gemini), auto-captioning (dual short/long) |
| `src/tools/describe-image.ts` | Image captioning tool (Gemini, OpenRouter), shared caption logic (dual short/long) |
| `src/tools/look-closer.ts` | Re-examine images for detailed descriptions after context fade |
| `src/llm/provider-presets.ts` | LLM provider types, connection profile types, and default presets |
| `src/llm/discord-settings.ts` | Discord settings type, load/save, token masking |
| `src/llm/home-settings.ts` | Home automation settings type, load/save (device list) |
| `src/llm/image-gen-settings.ts` | Image generator + captioning config type, load/save, masking |
| `src/llm/entity-core-settings.ts` | Entity-core LLM override settings type, load/save (model, temperature, maxTokens) |
| `src/tools/identity-helpers.ts` | Identity file utilities (XML parsing, MCP fallback, local snapshot restore) |
| `src/tools/identity-custom.ts` | Custom identity file tool (create, append, prepend, update_section, rewrite_section) |
| `src/memory/mod.ts` | Hierarchical memory system (daily summarization only; weekly/monthly/yearly consolidation moved to entity-core) |
| `src/rag/mod.ts` | RAG retrieval system |
| `src/mcp-client/mod.ts` | MCP client for entity-core connection |
| `src/lorebook/mod.ts` | Lorebook/world info system |
| `src/vault/mod.ts` | Data Vault — document storage and eager RAG |
| `src/db/schema.ts` | Database schema, migrations, vector table sync |
| `src/db/vector.ts` | sqlite-vec extension loading with auto-download from GitHub releases |
| `src/init/mod.ts` | Initialization — seeds identity, custom-tools, and vault template directories |
| `src/pulse/engine.ts` | Pulse system — autonomous scheduled entity prompts |
| `src/pulse/routes.ts` | Pulse API routes, CRUD, triggers, webhook endpoint |
| `src/pulse/templates.ts` | Pulse UI — settings hub card, editor, execution log |
| `src/pulse/timezone.ts` | Timezone conversion for local↔UTC Pulse scheduling |
| `src/tools/pulse-tools.ts` | Entity-facing Pulse tools (create, trigger, delete) |
| `src/push/mod.ts` | Push notification manager (VAPID keys, subscriptions, sending) |

## Situational Awareness

Real-time signal feeds injected into the entity's context every turn. Configured via Settings > Situational Awareness.

- **Current Conversation** — The conversation ID and title the entity is currently processing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zarilewis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
