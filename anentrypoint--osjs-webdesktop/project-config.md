---
trigger: always_on
description: Guidance for Claude Code when working with this repository.
---

# CLAUDE.md

Guidance for Claude Code when working with this repository.

## Project Overview

**vexify** is a portable vector database with semantic search, built on SQLite + embedding models (vLLM, Ollama, or Transformers.js). It processes multiple document formats (PDF, HTML, DOCX, JSON, CSV, XLSX), crawls websites, syncs Google Drive folders, and provides an MCP server for Claude Code integration.

**Core Philosophy**: Zero-config, local-first, CommonJS-compatible. No external APIs. Auto-detects vLLM (GPU-optimized) → Ollama (fallback) → transformers.js (in-process ONNX, no server required) → auto-installs Ollama.

## Architecture

### Core Components

**Storage** (`lib/adapters/sqlite.js`): SQLite + sqlite-vec extension, Float32 embeddings, JSON metadata

**Embedders** (`lib/embedders/`):
- `vllm.js`: OpenAI-compatible API, port 8000 (GPU-optimized, checked first)
- `ollama.js`: Ollama server, port 11434 (auto-installed fallback)
- `transformers.js`: In-process ONNX via @huggingface/transformers (no external server, ultimate fallback)
- Auto-detection order: vLLM → Ollama → transformers.js → auto-setup Ollama
- Default models: nomic-embed-text (Ollama/vLLM, 384-dim), Xenova/bge-base-en-v1.5 (transformers.js, 768-dim)

**Processors** (`lib/processors/`): BaseProcessor + format-specific (html, pdf, docx, excel, csv, json, txt), SHA-256 dedup

**Crawlers** (`lib/crawlers/`): web (Playwright depth/page limits), gdrive (incremental), code (semantic indexing)

**Search** (`lib/search/`): sqlite-vec (fast) + cosine (fallback)

**Utils**: embedding-queue (batching/retry), folder-sync (monitoring), ignore-manager (gitignore), ollama-setup (auto-install), pdf-embedder

**MCP** (`lib/mcp/server.js`): Model Context Protocol search tool, auto-syncs before queries, foreground only

**Module Structure**:
- Entry: `lib/index.js` (public API)
- CLI: `lib/bin/cli.js` (commands)
- Factories: VecStoreFactory (config/setup)
- Config: `lib/config/defaults.js` (centralized values)

### Data Flow

```
Input → Crawlers → Processors → Dedup → Embedding Queue → SQLite → Search
```

## Quick Start & Commands

### Development

```bash
# Local testing
npm link
vexify sync ./test.db ./documents
vexify query ./test.db "search term" 5
vexify crawl https://example.com --max-pages 50
vexify mcp --directory . --db-path ./.vexify.db

# Force specific embedder
vexify sync ./test.db ./documents --embedder-type vllm          # or ollama, transformers
vexify sync ./test.db ./documents --embedder-type transformers  # in-process ONNX (no server)

# vLLM (faster GPU inference)
vllm serve nomic-ai/nomic-embed-text-v1.5 --port 8000    # auto-detected first

# transformers.js (in-process, no server)
npm install @huggingface/transformers  # optional dependency, auto-detected third

# Debug
NODE_DEBUG=vexify vexify sync ./test.db ./documents

# Cleanup
npm unlink
```

### Publishing

```bash
# Update package.json version (major.minor.patch)
# Commit: "chore: bump version to X.Y.Z"
npm publish
# Use "fix:" prefix for bug fixes, "chore:" for version bumps
```

### Code Maintenance

```bash
find lib -name "*.js" -exec wc -l {} \; | sort -rn | head -10  # Find large files (>200 lines)
wc -l lib/**/*.js                                               # Check line counts
```

## Critical Constraints

**File Size**: Keep files <200 lines. Refactor candidates: `lib/utils/ignore-manager.js`, `lib/crawlers/web.js`

**Path Handling**: ALWAYS use `path.resolve()`, never string concatenation (CLI vs programmatic usage differs)

**Database Writes**: SQLite = one writer at a time. Multiple processes → deadlock. Each DB path exclusive to one process.

**Vector Dimensions**: Embeddings must match model (384 for nomic-embed-text, 768 for Xenova/bge-base-en-v1.5). Changes require migration in `lib/adapters/sqlite.js`. Don't mix embedders with different dimensions on same database.

**Memory Limits**: Playwright web crawler can OOM on large sites. Increase depth/page limits cautiously.

**Offline Environments**: Ollama/vLLM auto-pull models (2GB+). Pre-install: `ollama pull nomic-embed-text`. Alternatively, use transformers.js for in-process embeddings (no server, downloads models on first use).

**WSL Detection**: `lib/embedders/ollama.js:26` has 2000ms timeout to avoid blocking MCP startup

**Google Drive Quota**: Use `--incremental` flag for large folders (processes one file/call, stateless)

**Dedup**: SHA-256 content hashing in `lib/processors/dedup.js`. Same hash = duplicate regardless of source.

**Search Fallback**: Missing sqlite-vec → cosine similarity (10-100x slower, may timeout on large datasets)

**File Encoding**: UTF-8 assumed. Non-UTF-8 or PDFs with embedded fonts may fail extraction.

**Hardcoded Values**: Configuration in `lib/config/defaults.js`. Never inline batch sizes, retry counts, paths.

**Duplicate Logic**: `docx.js` and `excel.js` share patterns. Sync changes or extract shared code.

**Comments**: Replace with explicit names. If name is unclear, refactor, don't comment.

## Implementation Guide

### HTML Processing (jsdom 27.0.1 ES Module Issue)

`lib/processors/html.js` has two-stage fallback:
```javascript
try {
  const dom = new JSDOM(htmlContent, { url: options.url || 'http://localhost' });

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnEntrypoint/osjs-webdesktop](https://github.com/AnEntrypoint/osjs-webdesktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
