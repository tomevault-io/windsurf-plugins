---
trigger: always_on
description: > This file is for AI coding assistants (Cursor, Claude Code, Codex, GitHub Copilot) working on the ai-memory codebase. Read this first before touching code.
---

# AI Agent Instructions

> This file is for AI coding assistants (Cursor, Claude Code, Codex, GitHub Copilot) working on the ai-memory codebase. Read this first before touching code.

## Project Overview

ai-memory is a CLI tool + MCP server that extracts structured knowledge from AI editor conversations and saves it as **git-trackable Markdown files**. Targets Node.js >= 18.

- **npm package name:** `ai-memory-cli` (DO NOT rename — see [docs/decisions/2026-04-24-naming.md](docs/decisions/2026-04-24-naming.md))
- **Binary name:** `ai-memory`
- **GitHub owner:** `hyxnj666-creator`
- **Current version:** 2.5.0 (in tree; 2.4.0 last published on npm — see [ROADMAP.md](ROADMAP.md) and [CHANGELOG.md](CHANGELOG.md))
- **v2.5 features in tree:** `ai-memory try` (no-key demo), `rules --target skills` (Anthropic Skills), `--redact` at LLM call sites, Codex CLI as 5th editor, CCEB v1.1 F1 64.1%, LongMemEval-50 adapter, README "1M-context FAQ", dedup quality improvements
- **v2.6 features in tree (also not yet on npm):** `ai-memory link` (memory↔commit linking, weighted Jaccard scorer, `<!--links>` frontmatter), `init --schedule` (cross-platform daily cron: launchd/crontab/schtasks), single-chunk dedup fix + cross-type TODO subsumption (targets F1 75%+), dashboard graph enhancements (edge type differentiation, type filter toggles, hover highlight). Test suite **585**.
- **Before publishing v2.5.0:** read [`docs/v2.5-maintainer-handoff.md`](docs/v2.5-maintainer-handoff.md) — two maintainer-only tasks remain (v2.5-03 marketplace submissions + v2.5-07 AGENTS.md downstream eval).
- **Runtime deps:** `@modelcontextprotocol/sdk` (for `serve`) and `zod` (for bundle import validation). NO other runtime deps.

## Architecture

```
src/
├── index.ts              # CLI entry, sets emitWarning filter, dispatches to commands
├── cli.ts                # Argument parser (manual, no library)
├── types.ts              # ALL shared types live here
├── config.ts             # Loads .ai-memory/.config.json
├── public.ts             # Library entry (exports for programmatic use)
├── commands/             # One file per CLI command (14 commands)
│   ├── extract.ts        # Orchestrates extraction pipeline
│   ├── list.ts           # Lists conversations with status
│   ├── search.ts         # Keyword + hybrid search
│   ├── recall.ts         # Git-history time-travel retrieval (v2.4) — uses git/log-reader
│   ├── rules.ts          # Exports rules: Cursor .mdc and/or AGENTS.md (--target, v2.4)
│   ├── resolve.ts        # Marks memories resolved/active
│   ├── summary.ts        # LLM-generated project summary (supports --source-id / --convo)
│   ├── context.ts        # Generates continuation prompt (supports --source-id / --convo)
│   ├── init.ts           # Project initialization
│   ├── reindex.ts        # Rebuild embeddings; --dedup retroactive cleanup (v2.2)
│   ├── watch.ts          # Auto-extract on conversation changes
│   ├── dashboard.ts      # Launch local web UI
│   ├── export.ts         # Portable JSON bundle (v2.3)
│   ├── import.ts         # Import portable JSON bundle (v2.3)
│   ├── doctor.ts         # Health check (runtime/editors/LLM/store/embeddings/MCP, v2.4)
│   ├── try.ts            # No-API-key demo: bundled scenario → AGENTS.md inline (v2.5-02)
│   └── link.ts           # Scan git commits → auto-link to memories (weighted Jaccard, v2.6)
├── sources/              # Conversation parsers (one per editor)
│   ├── cursor.ts         # ~/.cursor/projects/*/agent-transcripts/
│   ├── claude-code.ts    # ~/.claude/projects/*/*.jsonl
│   ├── windsurf.ts       # Windsurf state.vscdb (SQLite via node:sqlite, Node 22+)
│   ├── copilot.ts        # VS Code chatSessions/*.json
│   └── detector.ts       # Auto-detects available sources
├── extractor/
│   ├── ai-extractor.ts   # Chunking, LLM calls, dedup, quality filter
│   ├── llm.ts            # OpenAI-compatible API client with retry + concurrency
│   └── prompts.ts        # All LLM prompts + buildDirectContext
├── embeddings/           # Semantic search (v2.0 phase 2)
│   ├── embed.ts          # Embedding API client
│   ├── vector-store.ts   # Flat-file .embeddings.json
│   ├── indexer.ts        # Auto-index on `remember`
│   └── hybrid-search.ts  # Semantic + keyword + time decay
├── mcp/                  # MCP server (v2.0 phase 1) + config writer (v2.4)
│   ├── server.ts         # stdio transport, handler wiring
│   ├── tools.ts          # remember / recall / search_memories tools
│   ├── resources.ts      # project-context resource
│   └── config-writer.ts  # idempotent merge into .cursor/mcp.json + .windsurf/mcp.json (init --with-mcp)
├── rules/                # Multi-target rules export (v2.4)
│   └── agents-md-writer.ts  # Pure idempotent merge into AGENTS.md (managed-section markers)
├── git/                  # Thin wrappers around the user's `git` binary (v2.4)
│   └── log-reader.ts     # parseGitLog / parseBulkLog (pure) + isGitRepo / getFileHistory / getRecentCommits / isPathTracked (execFile)
├── dashboard/            # Local web UI (v2.1)
│   ├── server.ts         # node:http API (/api/stats, /api/memories, /api/conversations, /api/quality, /api/graph)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyxnj666-creator/ai-memory](https://github.com/hyxnj666-creator/ai-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
