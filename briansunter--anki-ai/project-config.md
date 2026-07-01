---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Anki-AI** is an MCP server and CLI for Anki integration via Anki-Connect. It provides 96 tools across 8 categories for creating flashcards, managing reviews, analyzing learning data, and automating Anki workflows. Ships compiled JS via npm (`npx anki-ai`), uses Bun for development.

## Runtime & Commands

Use **Bun** (not Node.js) for all operations:

```bash
# Development
bun bin/anki-ai.ts mcp             # Start MCP server (Bun, fast)
bun run dev                       # Watch mode MCP server
bun run build                     # Compile to dist/ (tsc)
bun run typecheck                 # Type checking only

# CLI (development)
bun bin/anki-ai.ts deck list       # Use Bun entry point
bun bin/anki-ai.ts tools           # List all 96 tools
bun bin/anki-ai.ts run version     # Run any tool

# CLI (published, Node.js)
npx anki-ai deck list              # Uses compiled dist/main.js
npx anki-ai tools

# Testing
bun test                          # Run all tests
bun test tests/test-NAME.test.ts  # Run specific test file
bun test --watch                  # Watch mode
bun test:e2e                      # All E2E tests

# Code Quality
bun run lint                      # Run Biome linter
bun run lint:fix                  # Auto-fix linting issues

# Publishing (automated via release-please)
git commit -m "fix: description"  # Triggers patch release
git commit -m "feat: description" # Triggers minor release
git push                          # GitHub Actions publishes to npm
```

## Architecture

### Core Components

```
bin/anki-ai.ts              # Bun development entry point (#!/usr/bin/env bun)
src/
  main.ts                  # CLI entry point → compiles to dist/main.js (#!/usr/bin/env node)
  index.ts                 # MCP-only entry point (backward compat)
  shared/
    config.ts              # ANKI_CONNECT_URL, VERSION, DEBUG, debug()
    anki-connect.ts        # ankiConnect() + AnkiConnectError
    normalize.ts           # normalizeTags(), normalizeFields(), _encodeBase64()
    schema.ts              # zodToJsonSchema() (bug-sensitive, single while loop)
    types.ts               # ToolDef interface, AnkiConnectResponses
  tools/
    index.ts               # Aggregates all 96 tools from category files
    decks.ts (6)           # deckNames, createDeck, getDeckStats, etc.
    notes.ts (16)          # addNote, findNotes, notesInfo, updateNote, etc.
    cards.ts (19)          # findCards, getNextCards, cardsInfo, answerCards, etc.
    models.ts (9)          # modelNames, createModel, modelFieldNames, etc.
    media.ts (5)           # storeMediaFile, retrieveMediaFile, etc.
    stats.ts (7)           # getNumCardsReviewedToday, getDueCardsDetailed, etc.
    gui.ts (17)            # guiBrowse, guiAddCards, guiDeckReview, etc.
    system.ts (17)         # sync, exportPackage, multi, setDueDate, etc.
  cli/
    index.ts               # createProgram() factory with Commander.js
    run.ts                 # Generic: anki-ai run <tool> [json]
    tools-list.ts          # anki-ai tools [--category X] [--json]
    decks.ts, notes.ts, cards.ts, models.ts, stats.ts  # Category subcommands
  mcp/
    server.ts              # createServer() factory
    start.ts               # startMcpServer() with stdio transport
```

**Key design**: Tool handlers throw `AnkiConnectError` (not McpError). The MCP layer catches and wraps. CLI layer catches and prints.

Each tool has: `description`, `schema` (Zod), `handler` (async function). Smart features:
- **Auto-batching**: Operations >100 items split automatically
- **Pagination**: `findCards`, `findNotes`, `deckNames`, etc. support offset/limit
- **Queue priority**: `getNextCards` respects Learning > Review > New order
- **Smart normalization**: Tags/IDs work in any format

### Key Architectural Patterns

#### 1. Flexible Input Normalization
All tools accept multiple input formats for tags and IDs:
```typescript
// Tags can be:
["tag1", "tag2"]           // Array
"tag1 tag2"                // Space-separated string
'["tag1", "tag2"]'         // JSON string

// IDs can be:
[1234, 5678]               // Number array
["1234", "5678"]           // String array
```

#### 2. Pagination Pattern
```typescript
// Request:
{ query: "deck:current", offset: 0, limit: 100 }

// Response includes:
{
  cards: [...],
  pagination: {
    total: 500,
    offset: 0,
    limit: 100,
    hasMore: true,
    nextOffset: 100
  }
}
```

#### 3. Auto-batching Pattern
Operations automatically split when >100 items:
```typescript
// notesInfo with 250 IDs → splits into 3 requests of 100, 100, 50
await ankiConnect("notesInfo", { notes: [250 IDs] });
// Internally: batch1(100) + batch2(100) + batch3(50)
```

## Testing Architecture

### Test Categories

**tests/test-utils.ts** - Shared utilities:
- `ankiConnect()` - Direct Anki-Connect client with pagination support
- `setupTestEnvironment()` - Verifies Anki is running
- `createTestNotes()` - Batch create notes for testing
- `cleanupTestData()` - Remove test notes/decks

**Integration Tests** (require Anki running):
- `test-anki-connect.test.ts` - Basic connectivity, CRUD operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [briansunter/anki-ai](https://github.com/briansunter/anki-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
