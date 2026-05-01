---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm build      # Compile TypeScript → dist/
pnpm dev        # Watch mode with live reload (tsx watch)
pnpm start      # Run compiled binary (node dist/bin/axiom-wiki.js)
```

Tests: `pnpm test` (Vitest). Type-check with `pnpm build` (strict TypeScript compilation).

**Run `pnpm build` after every code change** to catch type errors early. The compiled output in `dist/` is what gets executed.

**Update docs after every user-facing change.** The documentation site lives in `docs/src/content/docs/` (Astro + Starlight). When adding or changing commands, features, or wiki structure, update the relevant doc pages under `docs/src/content/docs/commands/`, `docs/src/content/docs/guides/`, or `docs/src/content/docs/reference/`.

Run a specific CLI command in dev mode:
```bash
npx tsx bin/axiom-wiki.ts <command>   # e.g., status, query, ingest
```

## Architecture

Axiom Wiki is an AI-powered CLI wiki tool. The system has five main layers:

### Entry & Routing
- **`bin/axiom-wiki.ts`** — Commander.js CLI entry point defining 11 commands (`init`, `ingest`, `query`, `model`, `status`, `lint`, `watch`, `clip`, `sources`, `review`, `mcp`). Defaults to the home screen if no command is given.
- **`src/cli/index.tsx`** — Maps command types to Ink screen components (exhaustive type-checked dispatch).

### CLI/UI Layer (`src/cli/`)
- Built with **Ink 5 + React 18** (React for the terminal).
- **`screens/home.tsx`** — Main interactive REPL shell. Handles slash command autocomplete, keyboard navigation (↑↓ Tab Esc Ctrl+C), and routes to other screens.
- Each other screen (`ingest`, `query`, `init`, `watch`, `clip`, `sources`, `review`, `status`, `model`) is a self-contained Ink component.

### Agent Layer (`src/agent/`)
- **`index.ts`** — Creates a Mastra `Agent` with the resolved LLM model, system prompt, and tools.
- **`tools.ts`** — 14 Mastra tools (`read_page`, `write_page`, `list_pages`, `search_wiki`, `update_index`, `append_log`, `ingest_source`, `get_status`, `lint_wiki`, `list_sources`, `get_source`, `remove_source`, `get_contradictions`, `resolve_contradiction`).
- **`prompts.ts`** — System prompt defining wiki conventions (page frontmatter schema, category taxonomy, naming rules, cross-reference rules).

### Core/Wiki Layer (`src/core/`)
- **`wiki.ts`** — Atomic wiki I/O: `readPage`, `writePage`, `listPages`, `updateIndex`, `appendLog`, `getStatus`. Pages are Markdown with YAML frontmatter parsed by `gray-matter`.
- **`state.ts`** — Compilation state management. Tracks per-source SHA-256 hashes in `{wikiDir}/state.json` for incremental compilation. Key functions: `loadState`, `saveState`, `computeHash`, `detectChanges`, `recordIngest`, `migrateFromLog`.
- **`lock.ts`** — PID-based compilation lock (`{wikiDir}/lock`). Prevents concurrent ingest/compile operations. Stale locks from dead processes are auto-reclaimed. Key functions: `acquireLock`, `releaseLock`, `getLockInfo`, `forceReleaseLock`.
- **`files.ts`** — Normalizes source files into `SourceFile` objects. Supported: `.md`, `.txt`, `.pdf`, `.docx`, `.html`, `.png/.jpg/.jpeg/.webp`. PDF/images → base64; HTML → Markdown via `node-html-markdown`; DOCX → Markdown via `mammoth`.
- **`search.ts`** — Full-text search over title, summary, tags, and content with ranked results.
- **`sources.ts`** — Tracks ingested sources by parsing `wiki/log.md`.
- **`watcher.ts`** — Chokidar file watcher with `.axiomignore` support and debouncing.

### Config Layer (`src/config/`)
- **`index.ts`** — `conf`-based persistent config. Interface: `AxiomConfig { provider, apiKey, model, wikiDir, rawDir, ollamaBaseUrl }`.
- **`models.ts`** — Provider/model definitions for Google Gemini, OpenAI, Anthropic, and Ollama.

### MCP Layer (`src/mcp/server.ts`)
- Exposes all 14 agent tools as MCP resources via stdio transport for Claude Code/Cursor integration.
- Sets a global `isMcpMode` flag to suppress terminal output.

## Key Conventions

**Module system:** ESM (`"module": "NodeNext"` in tsconfig). Use `.js` extensions in relative imports even for `.ts` source files.

**Wiki directory structure** (wikiDir = `axiom` for local, user-chosen for global):
```
<wikiDir>/
├── config.json             ← Local project config (local scope only)
├── state.json              ← Compilation state (SHA-256 hashes, concept mappings)
├── map-state.json          ← Autowiki/sync state
├── lock                    ← PID-based lock (transient, present during ingest)
├── raw/                    ← Source files to ingest
│   └── .axiomignore
└── wiki/
    ├── pages/
    │   ├── entities/       ← People, places, orgs
    │   ├── concepts/       ← Ideas, theories
    │   ├── sources/        ← One summary per source
    │   └── analyses/       ← Comparisons, answers
    ├── index.md
    ├── log.md              ← Append-only operation log
    └── schema.md
```

**Page frontmatter schema:**
```yaml
---
title: "..."
summary: "..."
tags: [tag1, tag2]
category: entities | concepts | sources | analyses
sources: [file.pdf]
updatedAt: "YYYY-MM-DD"
---
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abubakarsiddik31/axiom-wiki](https://github.com/abubakarsiddik31/axiom-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
