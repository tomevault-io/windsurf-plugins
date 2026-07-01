---
trigger: always_on
description: Agent guidelines for working in the `adk-utils-go` repository.
---

# AGENTS.md

Agent guidelines for working in the `adk-utils-go` repository.

> Companion docs in `.agents/`: [DECISIONS.md](./DECISIONS.md) (per-provider
> design decisions and their rationale) and [TODOS.md](./TODOS.md) (deferred
> work and open review questions). Read DECISIONS.md before changing the
> `genai/*` adapters: several behaviours there are deliberate and load-bearing.

## Project Overview

A Go library providing utilities for Google's Agent Development Kit (ADK). This library extends ADK with additional backend implementations for topics like session management or memory services.

**Module**: `github.com/achetronic/adk-utils-go` (see `go.mod`)  
**Go Version**: 1.24.9+ (toolchain 1.25.5)  
**ADK Version**: v1.4.0

### Key Dependencies

| Package                        | Purpose                                            |
| ------------------------------ | -------------------------------------------------- |
| `google.golang.org/adk`        | Google ADK core framework                          |
| `google.golang.org/genai`      | Google GenAI types                                 |
| `github.com/redis/go-redis/v9` | Redis client for session storage                   |
| `github.com/lib/pq`            | PostgreSQL driver for memory storage               |
| `charm.land/catwalk`           | Embedded model registry (564 models, 23 providers) |

---

## Commands

### Build & Test

```bash
# Run all tests
go test ./...

# Run tests with verbose output
go test -v ./...

# Run tests for a specific package
go test -v ./memory/postgres/...
go test -v ./session/redis/...

# Run with race detection
go test -race ./...
```

### Module Management

```bash
# Download dependencies
go mod download

# Tidy dependencies
go mod tidy

# Verify dependencies
go mod verify
```

---

## Code Organization

```
adk-utils-go/
├── genai/
│   ├── openai/
│   │   └── openai.go            # OpenAI/Ollama-compatible LLM adapter
│   └── anthropic/
│       └── anthropic.go         # Anthropic Claude LLM adapter
├── session/
│   └── redis/
│       ├── session.go           # Redis-backed session.Service implementation
│       └── session_test.go      # Session tests (requires Redis)
├── memory/
│   ├── memorytypes/
│   │   └── types.go             # Shared types and interfaces (EntryWithID, ExtendedMemoryService)
│   └── postgres/
│       ├── memory.go            # PostgreSQL-backed memory.Service implementation
│       ├── memory_test.go       # Memory service tests (requires PostgreSQL)
│       ├── embedding.go         # OpenAI-compatible embedding model
│       └── embedding_test.go    # Embedding tests (uses httptest mocks)
├── artifact/
│   └── filesystem/
│       ├── artifact.go          # Filesystem-backed artifact.Service implementation
│       └── artifact_test.go     # Artifact tests
├── tools/
│   └── memory/
│       └── toolset.go           # Memory toolset for agent tools
├── plugin/
│   ├── contextguard/
│   │   ├── contextguard.go                        # Public API: New(), Add(), PluginConfig(), BeforeModel/AfterModel callbacks
│   │   ├── contextguard_unit_test.go               # 93 unit tests covering all functions + timing gap proofs
│   │   ├── compaction_strategy_multiturn_test.go   # 91 multi-turn session simulations (4k/8k/200k/1M, kube/coding/debug/storm patterns, tool defs, inline data, ratios, loops)
│   │   ├── compaction_strategy_singleshot_test.go  # Single-shot Compact() tests: kube-agent, mixed-debug, tool-storm, timing gap
│   │   ├── model_registry.go                       # ModelRegistry interface (ContextWindow, DefaultMaxTokens)
│   │   ├── model_registry_crush.go                 # CrushRegistry: catwalk embedded DB, 564 models, zero network
│   │   ├── compaction_utils.go                     # Internal helpers: state, summarization, tokens (contents + system + tools + inline data), calibration, splitting, continuation, todos, truncation
│   │   ├── compaction_strategy_threshold.go        # Token-threshold strategy (Crush-style full summary + hardening)
│   │   └── compaction_strategy_sliding_window.go   # Sliding-window strategy (turn-count, with recent tail + retry)
│   └── langfuse/
│       ├── langfuse.go      # Setup() API, spanEnricher (callbacks), enrichingExporter, enrichedSpan, helpers
│       ├── types.go         # Config struct with yaml/json tags, IsEnabled()
│       └── context.go       # Context helpers: WithUserID, WithTags, WithTraceMetadata, etc.
├── examples/
│   ├── openai-client/main.go
│   ├── anthropic-client/main.go
│   ├── session-memory/main.go
│   ├── long-term-memory/main.go
│   ├── full-memory/main.go
│   └── context-guard/main.go    # All 3 modes: CrushRegistry, WithMaxTokens, WithSlidingWindow
├── go.mod
└── go.sum
```

### Package Purposes

| Package               | Description                                                                                                       |
| --------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `genai/common`        | Helpers shared by the LLM adapters so cross-provider wire rules are implemented once (e.g. `MarshalToolPayload`)  |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [achetronic/adk-utils-go](https://github.com/achetronic/adk-utils-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
