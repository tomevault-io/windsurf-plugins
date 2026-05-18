---
trigger: always_on
description: > Context for AI agents (Claude Code, Copilot, Cursor, etc.) working on this codebase. Read this fully before making changes.
---

# AGENTS.md — Bifrost AI Gateway

> Context for AI agents (Claude Code, Copilot, Cursor, etc.) working on this codebase. Read this fully before making changes.

## What is Bifrost?

Bifrost is a high-performance AI gateway that unifies 20+ LLM providers behind a single OpenAI-compatible API with ~11µs overhead at 5,000 RPS. It also serves as an MCP (Model Context Protocol) gateway, turning static chat models into tool-calling agents.

GitHub: `maximhq/bifrost`

---

## Repository Layout

```
bifrost/
├── core/                           # Go core library — the engine
│   ├── bifrost.go                  # Main struct, request queuing, provider lifecycle (~3.4K lines)
│   ├── inference.go                # Inference routing, fallbacks, streaming dispatch (~1.9K lines)
│   ├── mcp.go                     # MCP integration entry point
│   ├── schemas/                   # ALL shared Go types — 41 files
│   │   ├── bifrost.go             # BifrostConfig, ModelProvider enum, RequestType enum, context keys
│   │   ├── provider.go            # Provider interface (30+ methods), NetworkConfig, ProviderConfig
│   │   ├── plugin.go              # LLMPlugin, MCPPlugin, HTTPTransportPlugin, ObservabilityPlugin
│   │   ├── context.go             # BifrostContext (custom context.Context with mutable values)
│   │   ├── chatcompletions.go     # Chat completion request/response types
│   │   ├── responses.go           # OpenAI Responses API types
│   │   ├── embedding.go           # Embedding types
│   │   ├── images.go              # Image generation types
│   │   ├── batch.go               # Batch operation types
│   │   ├── files.go               # File management types
│   │   ├── mcp.go                 # MCP types
│   │   ├── trace.go               # Tracer interface
│   │   └── logger.go              # Logger interface
│   ├── providers/                 # 20+ provider implementations
│   │   ├── openai/                # Reference implementation (largest, most complete)
│   │   ├── anthropic/             # Non-OpenAI-compatible example
│   │   ├── bedrock/               # AWS event-stream protocol
│   │   ├── gemini/                # Google-specific API shape
│   │   ├── groq/                  # OpenAI-compatible (minimal, delegates to openai/)
│   │   └── utils/                 # Shared: HTTP client, SSE parsing, error handling, scanner pool
│   ├── pool/                      # Generic Pool[T] — dual-mode (prod: sync.Pool, debug: full tracking)
│   │   ├── pool_prod.go           # Zero-overhead sync.Pool wrapper (default build)
│   │   └── pool_debug.go          # Double-release/use-after-release/leak detection (-tags pooldebug)
│   ├── mcp/                       # MCP protocol implementation
│   │   ├── agent.go               # Agent orchestration loop (multi-turn tool calling)
│   │   ├── clientmanager.go       # MCP client lifecycle management
│   │   ├── toolmanager.go         # Tool registration, discovery, filtering
│   │   ├── healthmonitor.go       # Client health monitoring
│   │   └── codemode/starlark/     # Starlark sandbox for code-mode execution
│   └── internal/
│       ├── llmtests/              # LLM integration test infra (48 files, scenario-based)
│       └── mcptests/              # MCP/Agent test infra (40+ files, mock-based)
│
├── framework/                     # Data persistence, streaming, ecosystem utilities
│   ├── configstore/               # Config storage backends (file, postgres)
│   ├── logstore/                  # Log storage backends (file, postgres)
│   ├── vectorstore/               # Vector storage (Weaviate, Qdrant, Redis, Pinecone)
│   ├── streaming/                 # Streaming accumulator, delta copying, response marshaling
│   │   ├── accumulator.go         # Chunk accumulation into full response (~24KB)
│   │   ├── chat.go                # Chat stream handling (~17KB)
│   │   └── responses.go           # Response stream marshaling (~35KB)
│   ├── modelcatalog/              # Model metadata registry
│   ├── tracing/                   # Distributed tracing helpers
│   └── encrypt/                   # Encryption utilities
│
├── transports/
│   ├── config.schema.json         # JSON Schema — THE source of truth for config.json (~2700 lines)
│   └── bifrost-http/              # HTTP gateway transport
│       ├── server/                # Server lifecycle, route registration
│       ├── handlers/              # 27 HTTP endpoint handlers
│       │   ├── inference.go       # Chat/text completions, responses API (~109KB)
│       │   ├── mcpinference.go    # MCP tool execution
│       │   ├── governance.go      # Virtual keys, teams, customers, budgets (~100KB)
│       │   ├── providers.go       # Provider CRUD, key management
│       │   ├── mcp.go             # MCP client registry management
│       │   ├── logging.go         # Log queries, stats, histograms
│       │   ├── config.go          # System configuration
│       │   ├── plugins.go         # Plugin CRUD
│       │   ├── cache.go           # Cache management
│       │   ├── session.go         # Auth/session management
│       │   ├── health.go          # Health checks
│       │   ├── mcpserver.go       # MCP server (SSE/streamable HTTP)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maximhq/bifrost](https://github.com/maximhq/bifrost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
