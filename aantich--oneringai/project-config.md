---
trigger: always_on
description: **Name**: `@everworker/oneringai`
---

# Claude Development Guide

## Project Overview

**Name**: `@everworker/oneringai`
**Purpose**: Unified AI agent library with multi-vendor support for text, image, video, audio, and agentic workflows
**Language**: TypeScript (strict mode) | **Runtime**: Node.js 18+ | **Package**: ESM

## Architecture: Connector-First Design

```
User Code → Connector Registry → Agent → Provider Factory → ITextProvider
```

**Key Principles:**
1. **Connectors are single source of truth** for auth - no dual systems
2. **Named connectors** - multiple keys per vendor (`openai-main`, `openai-backup`)
3. **Explicit vendor** - uses `Vendor` enum, no auto-detection
4. **Unified tool management** - `agent.tools === agent.context.tools` (same instance)

## Core Classes

- **Connector** (`src/core/Connector.ts`) — Static auth registry. `Connector.create()` / `Connector.get()`
- **Agent** (`src/core/Agent.ts`) — Main agent extending BaseAgent. `Agent.create({ connector, model, tools })`
- **AgentContextNextGen** (`src/core/context-nextgen/AgentContextNextGen.ts`) — Plugin-first context manager
- **ToolManager** (`src/core/ToolManager.ts`) — Unified tool management + execution. `IToolExecutor`, `IDisposable`. Per-tool circuit breakers
- **Vendor** (`src/core/Vendor.ts`) — Const object: `{ OpenAI, Anthropic, Google, GoogleVertex, Groq, Together, Grok, DeepSeek, Mistral, Perplexity, Ollama, Custom }`

## AgentContextNextGen

Plugin-first context manager. Features enable/disable plugins:

```typescript
interface ContextFeatures {
  workingMemory?: boolean;         // default: true
  inContextMemory?: boolean;       // default: true
  persistentInstructions?: boolean; // default: false
  userInfo?: boolean;              // default: false
  toolCatalog?: boolean;           // default: false
  sharedWorkspace?: boolean;       // default: false
}
```

**Tool Catalog scoping:** `toolCategories` = built-in, `identities` = connector categories, `pinned` = always loaded. Plugin tools always available.

**Key APIs:** `ctx.tools`, `ctx.memory`, `ctx.features`, `ctx.registerPlugin()`, `ctx.getPlugin<T>(name)`, `ctx.addUserMessage()`, `ctx.addAssistantResponse()`, `ctx.addToolResults()`, `ctx.prepare()`, `ctx.save()`, `ctx.load()`

**Compaction:** Happens once before LLM call via `StrategyRegistry`. Default: algorithmic (75% threshold). Tool pairs always removed together.

## Agent run() / stream()

`RunOptions`: `thinking` (vendor-agnostic), `temperature`, `vendorOptions` — override per call.

**Direct LLM access:** `agent.runDirect()` / `agent.streamDirect()` — bypasses context management. Options: `instructions`, `includeTools`, `temperature`, `maxOutputTokens`, `responseFormat`, `thinking`, `vendorOptions`

## Directory Structure

```
src/
├── index.ts                    # Main exports (~300 items)
├── core/                       # Agent, BaseAgent, Connector, Vendor, ToolManager, constants
│   ├── context-nextgen/        # AgentContextNextGen + plugins
│   ├── orchestrator/           # createOrchestrator, orchestration tools
│   ├── permissions/            # PermissionPolicyManager, policies, UserPermissionRulesEngine
│   ├── mcp/                    # MCPClient, MCPRegistry
│   └── StorageRegistry.ts      # Centralized storage backend registry
├── domain/
│   ├── entities/               # Model.ts, Tool.ts, Message.ts, Memory.ts, Services.ts (35+)
│   ├── interfaces/             # ITextProvider, IAudioProvider, IToolExecutor, IDisposable, IContextStorage
│   └── errors/                 # AIErrors.ts, MCPError.ts
├── capabilities/               # search/, scrape/, images/, video/
├── infrastructure/
│   ├── providers/              # OpenAI, Anthropic, Google, Generic (+ base/)
│   ├── resilience/             # CircuitBreaker, BackoffStrategy, RateLimiter
│   └── storage/                # FileContextStorage, InMemoryStorage
├── tools/                      # filesystem/, shell/, web/, desktop/, connector/, code/, json/
├── connectors/                 # oauth/, storage/
└── utils/
```

## Unified Store Tools

All CRUD plugins share 5 generic tools routed by `StoreToolsManager`:

| Tool | Purpose |
|------|---------|
| `store_get(store, key?)` | Get entry or all entries |
| `store_set(store, key, value, ...)` | Create/update entry |
| `store_delete(store, key)` | Delete entry |
| `store_list(store, options?)` | List with optional filtering |
| `store_action(store, action, params?)` | Store-specific ops |

**Store IDs:** `"memory"`, `"context"`, `"instructions"`, `"user_info"`, `"workspace"` (+ custom `IStoreHandler` plugins)

**Custom CRUD plugin:** Implement `IContextPluginNextGen` + `IStoreHandler` with `storeId`/`storeDescription` + handle methods. Register via `ctx.registerPlugin()` — auto-detected.

## NextGen Plugins

| Plugin | Store ID | Purpose |
|--------|----------|---------|
| **WorkingMemoryPluginNextGen** | `"memory"` | Tiered storage (raw/summary/findings), auto-eviction. `ctx.memory` shortcut |
| **InContextMemoryPluginNextGen** | `"context"` | KV storage **directly in context** (no retrieval needed). Priority-based eviction (low→high, critical never evicted) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aantich/oneringai](https://github.com/aantich/oneringai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
