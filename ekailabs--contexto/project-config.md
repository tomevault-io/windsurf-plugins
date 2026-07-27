---
trigger: always_on
description: > Guide for AI agents working on the memory service codebase.
---

# AGENT.md — Memory Service (Ekai)

> Guide for AI agents working on the memory service codebase.

## 1. Overview

The memory service is a **neuroscience-inspired cognitive memory system** that runs as a standalone Express server (default port `4005`). It provides three memory sectors — episodic, semantic, and procedural — with PBWM-inspired scoring for retrieval gating and semantic fact consolidation for knowledge graph maintenance.

### Key Features
- **3 memory sectors**: episodic (events), semantic (knowledge graph triples), procedural (step-by-step workflows)
- **PBWM gating**: Prefrontal–basal-ganglia-inspired scoring with relevance, expected value, control signal, and noise
- **Semantic consolidation**: Merge/supersede/insert logic for knowledge graph facts using embedding similarity
- **Working memory cap**: Gate threshold of 0.5, hard cap of 8 items
- **Multi-profile support**: Isolated memory spaces per profile slug
- **Dual provider support**: Gemini (default) and OpenAI for embeddings and extraction

### Architecture Summary

```
POST /v1/ingest           → extract(LLM) → 3 sectors → embed → SQLite
POST /v1/ingest/documents → read .md files → chunk → extract → dedup → store with source
POST /v1/search           → embed query → brute-force cosine → PBWM gate → working memory (cap 8)
GET  /v1/graph/*          → BFS traversal over semantic_memory triples
```

---

## 2. Quick Reference

### Directory Structure

```
memory/
├── package.json           # ESM package; scripts: build, start, prestart
├── tsconfig.json          # ESNext target, strict: false, output → ./dist
├── memory.db              # SQLite database (runtime artifact, gitignored)
├── README.md              # User-facing documentation
├── AGENT.md               # This file
└── src/
    ├── index.ts            # Barrel re-export of all modules
    ├── server.ts           # Express app, all route definitions, env loading
    ├── sqlite-store.ts     # Core storage: schema, ingest, query, CRUD
    ├── documents.ts        # Document ingestion: markdown chunking + orchestration
    ├── types.ts            # All TypeScript interfaces and type aliases
    ├── scoring.ts          # PBWM gate scoring algorithm
    ├── wm.ts               # Working memory filter and cap logic
    ├── consolidation.ts    # Semantic fact consolidation (merge/supersede/insert)
    ├── semantic-graph.ts   # Graph traversal (BFS paths, neighbors, reachability)
    ├── utils.ts            # cosine similarity, sigmoid, gaussian noise, profile slug
    └── providers/
        ├── registry.ts     # Provider config, URL builder, auth for Gemini/OpenAI
        ├── embed.ts        # Embedding API caller
        ├── extract.ts      # LLM-based memory extraction (structured JSON)
        └── prompt.ts       # System prompt for the extraction LLM
```

### Build & Run

```bash
# Install dependencies (from repo root)
npm install -w memory

# Build TypeScript
npm run build -w memory

# Start server (runs prestart → build automatically)
npm start -w memory

# Run all services together (gateway + dashboard + memory)
npm run dev:all
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `GOOGLE_API_KEY` | *required* | Gemini API key for extraction and embeddings |
| `MEMORY_PORT` | `4005` | HTTP server port |
| `MEMORY_DB_PATH` | `./memory.db` | SQLite database file path |
| `MEMORY_CORS_ORIGIN` | `*` | Comma-separated allowed CORS origins |
| `MEMORY_EMBED_PROVIDER` | `gemini` | Embedding provider: `gemini` or `openai` |
| `MEMORY_EXTRACT_PROVIDER` | `gemini` | Extraction provider: `gemini` or `openai` |
| `GEMINI_EMBED_MODEL` | `gemini-embedding-001` | Gemini embedding model |
| `GEMINI_EXTRACT_MODEL` | `gemini-2.5-flash` | Gemini extraction model |
| `OPENAI_API_KEY` | — | Required if using OpenAI provider |
| `OPENAI_EMBED_MODEL` | `text-embedding-3-small` | OpenAI embedding model |
| `OPENAI_EXTRACT_MODEL` | `gpt-4o-mini` | OpenAI extraction model |

Env files are loaded from `memory/.env` first, then root `.env` (see `server.ts:7-8`).

### API Endpoints at a Glance

| Method | Path | Purpose |
|--------|------|---------|
| `GET` | `/health` | Health check |
| `GET` | `/v1/profiles` | List all profiles |
| `DELETE` | `/v1/profiles/:slug` | Delete a profile and all its memories |
| `POST` | `/v1/ingest` | Ingest experience → extract → embed → store |
| `POST` | `/v1/ingest/documents` | Ingest markdown files from a directory with dedup |
| `POST` | `/v1/search` | Search memories with PBWM-gated retrieval |
| `GET` | `/v1/summary` | Per-sector counts + recent items |
| `PUT` | `/v1/memory/:id` | Update a memory's content |
| `DELETE` | `/v1/memory/:id` | Delete a single memory |
| `DELETE` | `/v1/memory` | Delete all memories for a profile |
| `DELETE` | `/v1/graph/triple/:id` | Delete a semantic triple |
| `GET` | `/v1/graph/triples` | Query triples by entity |
| `GET` | `/v1/graph/neighbors` | Get connected entities |
| `GET` | `/v1/graph/paths` | BFS path-finding between entities |
| `GET` | `/v1/graph/visualization` | Graph data for UI rendering |

---

## 3. Core Concepts

### 3a. Three Memory Sectors


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ekailabs/contexto](https://github.com/ekailabs/contexto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
