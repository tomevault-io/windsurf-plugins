---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**MCP AutoMem** is an MCP (Model Context Protocol) server that bridges AI assistants like Claude with the AutoMem memory service. It enables AI to store, recall, and associate memories using FalkorDB (graph database) and Qdrant (vector search).

**Core Purpose:**
- Translate MCP tool calls into AutoMem API requests
- Provide memory management for AI assistants (storage, hybrid search, relationships)
- Support Claude Code integration with automatic session capture hooks

## Build & Development

```bash
# Build TypeScript to dist/
npm run build

# Development with hot-reload
npm run dev

# Test server help output
npm test

# Publish preparation
npm run prepublishOnly  # Runs build automatically
```

## Commit Standards (Required)

This repo uses **Conventional Commits** so Release Please can generate releases reliably.

Accepted examples:
```text
fix: prevent stdout corruption in stdio mode
feat: add cursor setup command
chore: update dependencies
docs: clarify Claude Desktop config
```

Notes:
- PR titles must be Conventional Commit format because we squash-merge and the PR title becomes the merge commit message.
- A git `commit-msg` hook (Husky + Commitlint) is included to catch mistakes locally, and CI enforces PR titles.

## Key Commands

### CLI Commands (via npx or global install)

```bash
# Guided setup wizard (creates .env, prints config snippets)
npx @verygoodplugins/mcp-automem setup

# Install Claude Code automation hooks & merge settings
npx @verygoodplugins/mcp-automem claude-code
npx @verygoodplugins/mcp-automem claude-code --profile lean      # Quiet defaults (recommended)
npx @verygoodplugins/mcp-automem claude-code --profile extras    # More hooks + status line
npx @verygoodplugins/mcp-automem claude-code --dry-run           # Preview changes
npx @verygoodplugins/mcp-automem claude-code --dir <path>        # Custom target directory

# Print config snippets for Claude Desktop/Cursor/Code
npx @verygoodplugins/mcp-automem config --format=json

# Process memory queue manually (normally automatic via Stop hook)
npx @verygoodplugins/mcp-automem queue
npx @verygoodplugins/mcp-automem queue --file ~/.claude/scripts/memory-queue.jsonl
```

### Testing MCP Server Locally

```bash
# Start the MCP server directly (stdio mode)
node dist/index.js

# Or via npm start
npm start
```

The server expects `AUTOMEM_API_URL` (and optionally `AUTOMEM_API_KEY`) in environment or `.env` file. `AUTOMEM_ENDPOINT` is the deprecated alias and is still read as a fallback.

## Architecture

```
┌─────────────────────────────────────────┐
│  MCP Client (Claude Desktop/Code/Cursor) │
│  - Calls MCP tools via stdio/SSE        │
└─────────────┬───────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────┐
│  MCP AutoMem Server (this TypeScript app)│
│  - src/index.ts: MCP Server setup       │
│  - src/automem-client.ts: API client    │
│  - src/cli/*: Setup/queue/config CLIs   │
│  - Translates MCP calls → HTTP API      │
└─────────────┬───────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────┐
│  AutoMem Service (Python Flask API)     │
│  - /memory (POST/PATCH/DELETE)          │
│  - /recall (GET with hybrid search)     │
│  - /associate (POST relationships)      │
│  - /health (GET status)                 │
│  - Stores in FalkorDB + Qdrant          │
└──────────────────────────────────────────┘
```

### Code Organization

```
src/
├── index.ts              # MCP server entry point, tool registration, handlers
├── automem-client.ts     # HTTP client for AutoMem API
├── types.ts              # TypeScript interfaces (configs, args, results)
└── cli/
    ├── setup.ts          # Setup wizard (creates .env, prints config)
    ├── claude-code.ts    # Claude Code hook installation
    ├── queue.ts          # Memory queue processor (drains queue → AutoMem)
    └── templates.ts      # Config snippet generation

templates/
├── claude-code/
│   ├── hooks/            # PostToolUse and Stop hook scripts
│   ├── scripts/          # Memory processing, filters, notifications
│   ├── settings.json     # Default hook config (merged into ~/.claude/settings.json)
│   └── profiles/
│       ├── settings.lean.json    # Quiet profile (recommended)
│       └── settings.extras.json  # Full-featured profile (optional)
├── CLAUDE_CODE_INTEGRATION.md   # Complete hook system documentation
└── CLAUDE_MD_MEMORY_RULES.md    # Memory rules template for ~/.claude/CLAUDE.md
```

## MCP Tools

The server exposes 6 tools to AI assistants. Several are mode-multiplexed — the mode is selected by which params you pass, so the tool count stays small while the surface area covers all of AutoMem's memory CRUD endpoints.

1. **store_memory** — Two modes:
   - **Single (default):** `content` plus optional `tags`, `importance`, `metadata`, `type`, `confidence`, `embedding`, `t_valid`, `t_invalid`, `id`.
   - **Batch:** `memories: [...]` (≤500 items) for bulk ingestion. Per-item `id`, `embedding`, `t_valid`, `t_invalid` are NOT supported in batch mode — use single-mode for those.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [verygoodplugins/mcp-automem](https://github.com/verygoodplugins/mcp-automem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
