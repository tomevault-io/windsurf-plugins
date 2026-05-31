---
trigger: always_on
description: **Last Updated**: 2026-04-21
---

# CLAUDE.md - AI Assistant Project Guide

**Last Updated**: 2026-04-21
**Target**: AI assistants (Claude Code, GitHub Copilot, Cursor, etc.)
**Current release**: see [GitHub Releases](https://github.com/aixgo-dev/aixgo/releases/latest) (canonical version is the git tag)

Quick reference for AI assistants working with Aixgo - a production-grade AI agent framework for Go.

## Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Code Conventions](#code-conventions)
- [Key Concepts](#key-concepts)
- [Website](#website)
- [Common Tasks](#common-tasks)
- [Quick Reference](#quick-reference)

---

## Project Overview

Aixgo is a **production-grade AI agent framework for Go** enabling secure, scalable multi-agent systems without Python dependencies.

**Installation**: `go get github.com/aixgo-dev/aixgo`

### Why Aixgo?

| Metric | Aixgo | Python Frameworks |
|--------|-------|-------------------|
| Binary Size | <20MB | 1GB+ containers |
| Cold Start | <100ms | 10-45s |
| Concurrency | True parallelism (no GIL) | GIL-limited |
| Type Safety | Compile-time | Runtime errors |

### Key Capabilities

- **13 orchestration patterns** - Supervisor, Sequential, Parallel, Router, Swarm, Hierarchical, RAG, Reflection, Ensemble, Classifier, Aggregation, Planning, MapReduce
- **6 agent types** - ReAct, Classifier, Aggregator, Planner, Producer, Logger
- **8+ LLM providers** - OpenAI, Anthropic, Gemini, xAI, Vertex AI, Amazon Bedrock, HuggingFace, + inference services (Ollama, vLLM)
- **Validation retry** - Structured output validation with automatic retry (40-70% improved reliability)
- **MCP support** - Model Context Protocol for tool calling (local, gRPC, multi-server)

### Target Users

- **Backend Engineers** - Building AI-powered services in Go stacks
- **DevOps Teams** - Deploying production AI systems with minimal footprint
- **Data Engineers** - Adding AI enrichment to ETL pipelines
- **Enterprises** - Running AI agents on-premises or edge devices

### Current Status

- **Current release**: see [GitHub Releases](https://github.com/aixgo-dev/aixgo/releases/latest) — the git tag is the single source of truth
- **Maturity**: Production-ready for core features
- **Go Version**: 1.26+
- **License**: MIT

### Development Focus

Current priorities:
1. **Stability** - Production hardening and edge case handling
2. **Observability** - Enhanced tracing, metrics, and cost tracking
3. **Patterns** - Expanding orchestration pattern capabilities
4. **MCP** - Multi-server support and tool discovery

### Core Features Reference

**MANDATORY**: See **[docs/FEATURES.md](docs/FEATURES.md)** for the complete authoritative feature catalog (200+ features).

**Update Requirement**: When making changes, **MUST** update `docs/FEATURES.md` to reflect:
- New/modified/deprecated features
- Status changes (🚧 In Progress → ✅ Implemented)
- New configuration options or examples

**Status Indicators**:
- ✅ Implemented
- 🚧 In Progress
- 🔮 Roadmap
- ❌ Not Available

---

## Architecture

### Layer Overview

```text
Application (YAML Config, CLI, Examples)
    ↓
Orchestration (Supervisor, Patterns, Workflow)
    ↓
Agents (ReAct, Classifier, Aggregator, Planner, Producer, Logger)
    ↓
Runtime (Local: Go channels, Distributed: gRPC)
    ↓
Integration (LLM Providers, MCP, Vector Stores, Embeddings)
    ↓
Observability & Security (OpenTelemetry, Auth, Rate Limiting)
```

### Runtime Systems

**Local Runtime** (`runtime.go`):
- In-process communication via Go channels
- Single binary deployment
- Use: `NewRuntime()`

**Distributed Runtime** (`internal/runtime/`):
- Multi-node orchestration via gRPC
- Horizontal scaling across machines
- Protocol buffers in `proto/`

### Key Package Map

**Core**:
- `aixgo.go` - Entry point, config loading
- `runtime.go` - Local runtime (Runtime)

**Agents** (`agents/`):
- `react.go` - LLM + tool calling
- `classifier.go` - Content classification
- `aggregator.go` - Multi-agent synthesis
- `planner.go` - Task planning
- `producer.go` / `logger.go` - Message generation/logging

**Internal**:
- `internal/agent/` - Factory, types, interfaces
- `internal/llm/provider/` - OpenAI, Anthropic, Gemini, xAI, Vertex, HuggingFace
- `internal/llm/validator/` - Structured output validation retry
- `internal/supervisor/` - Orchestration core
- `internal/supervisor/patterns/` - Parallel, MapReduce, etc.
- `internal/observability/` - OpenTelemetry
- `internal/runtime/` - Distributed runtime (gRPC)

**Public** (`pkg/`):
- `pkg/mcp/` - Model Context Protocol
- `pkg/vectorstore/` - Firestore, Memory
- `pkg/embeddings/` - OpenAI, HuggingFace
- `pkg/security/` - Auth, rate limiting, SSRF, sanitization
- `pkg/observability/` - Health checks, metrics

**Other**:
- `proto/` - Protocol buffers
- `cmd/` - CLI tools (orchestrator, benchmark, deploy)
- `config/` - Example YAML configs
- `examples/` - 15+ production examples
- `docs/` - Comprehensive documentation

---

## Code Conventions

### Go Standards

Follow [Effective Go](https://golang.org/doc/effective_go.html) and [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments).

### Naming

```go
// Interfaces: Noun/adjective
type Agent interface { ... }

// Constructors: New* prefix
func NewRuntime() *Runtime

// Factory: Create* prefix

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aixgo-dev/aixgo](https://github.com/aixgo-dev/aixgo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
