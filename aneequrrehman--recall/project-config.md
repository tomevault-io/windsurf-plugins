---
trigger: always_on
description: This file provides guidance for AI assistants working with the Recall codebase.
---

# CLAUDE.md

This file provides guidance for AI assistants working with the Recall codebase.

## Project Overview

Recall is a memory layer for AI applications that runs inside your stack. Unlike hosted memory services, Recall stores memories in your existing database with zero vendor lock-in. It provides:

- LLM-powered fact extraction from conversations
- Intelligent memory consolidation (ADD/UPDATE/DELETE/NONE decisions)
- Vector similarity search using embeddings
- Pluggable architecture for databases, embeddings, and extractors

## Repository Structure

This is a **pnpm monorepo** using **Turborepo** for build orchestration.

```
recall/
├── packages/           # Published npm packages
│   ├── core/           # @youcraft/recall - Core memory client
│   ├── adapter-sqlite/ # @youcraft/recall-adapter-sqlite
│   ├── adapter-postgresql/ # @youcraft/recall-adapter-postgresql
│   ├── adapter-mysql/  # @youcraft/recall-adapter-mysql
│   ├── embeddings-openai/ # @youcraft/recall-embeddings-openai
│   ├── embeddings-cohere/ # @youcraft/recall-embeddings-cohere
│   ├── extractor-openai/ # @youcraft/recall-extractor-openai
│   ├── extractor-anthropic/ # @youcraft/recall-extractor-anthropic
│   ├── ai-sdk/         # @youcraft/recall-ai-sdk - Vercel AI SDK integration
│   ├── mcp/            # @youcraft/recall-mcp - MCP tool definitions
│   ├── mcp-server/     # @youcraft/recall-mcp-server - MCP server
│   └── recall-structured/ # @youcraft/recall-structured - Schema-based memory
├── apps/
│   └── web/            # Next.js dashboard app with auth & API
├── examples/
│   ├── with-inngest/   # Inngest background extraction example
│   ├── with-inngest-structured/ # Structured memory with Inngest
│   └── with-wdk/       # Vercel Workflow DevKit example
├── docs/               # Astro documentation site
├── package.json        # Root package.json with workspace scripts
├── pnpm-workspace.yaml # Workspace configuration
├── turbo.json          # Turborepo task configuration
└── vitest.workspace.ts # Vitest workspace configuration
```

## Key Packages

### Core (`packages/core`)

The main memory client. Exports:

- `createMemory(config)` - Factory function returning a memory client
- `MemoryClient` - Type for the returned client
- `inMemoryAdapter` - In-memory adapter for testing
- Core types: `Memory`, `DatabaseAdapter`, `EmbeddingsProvider`, `ExtractorProvider`

### Database Adapters

All adapters implement `DatabaseAdapter` interface:

- `adapter-sqlite` - Uses `better-sqlite3`, stores embeddings as JSON
- `adapter-postgresql` - Uses `pg`, designed for pgvector
- `adapter-mysql` - Uses `mysql2`

### Embeddings Providers

All implement `EmbeddingsProvider` interface:

- `embeddings-openai` - OpenAI text-embedding models
- `embeddings-cohere` - Cohere embed models

### Extractor Providers

All implement `ExtractorProvider` interface with optional `consolidate()`:

- `extractor-openai` - GPT-based fact extraction
- `extractor-anthropic` - Claude-based fact extraction

### AI SDK Integration (`packages/ai-sdk`)

Wraps Vercel AI SDK models with memory capabilities:

```typescript
const recall = createRecall({ memory, onExtract })
const model = recall(anthropic('claude-sonnet-4-20250514'), { userId })
```

### Structured Memory (`packages/recall-structured`)

Schema-based memory with Zod validation. Supports:

- Pure classification
- Data extraction
- Intent processing (query/insert/update/delete)
- Agent with CRUD tools

## Core Memory Extraction & Processing

### Overview

The core memory system uses a two-phase LLM approach:

1. **Extraction** - Extract discrete facts from conversation text
2. **Consolidation** - For each fact, decide how to merge with existing memories

### Extraction Phase

When `memory.extract(text, { userId })` is called:

```
Input Text → LLM Extraction → Array of Facts
```

The extractor uses structured output (Zod schema) to extract atomic facts:

```typescript
// Extraction prompt focuses on:
// - Personal preferences, facts, relationships, goals
// - User's name (high priority)
// - Persistent information (not transient queries)
// - Third-person format: "User's name is John", "User lives in NYC"

const extracted = await extractor.extract(text)
// Returns: [{ content: "User likes TypeScript" }, { content: "User works at Acme" }]
```

### Consolidation Phase

For each extracted fact, the system prevents duplicates:

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  New Fact       │────▶│  Embed & Search  │────▶│  Similar        │
│  "Name is John" │     │  (top 5)         │     │  Memories       │
└─────────────────┘     └──────────────────┘     └────────┬────────┘
                                                          │
                        ┌──────────────────┐              │
                        │  UUID → Integer  │◀─────────────┘
                        │  Mapping         │
                        └────────┬─────────┘
                                 │
                        ┌────────▼─────────┐
                        │  LLM Decision    │
                        │  ADD/UPDATE/     │
                        │  DELETE/NONE     │
                        └────────┬─────────┘
                                 │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aneequrrehman/recall](https://github.com/aneequrrehman/recall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
