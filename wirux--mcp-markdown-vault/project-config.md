---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Headless, Dockerized TypeScript MCP (Model Context Protocol) server for markdown-based knowledge bases (Obsidian, Logseq, Dendron, Foam, or any folder of `.md` files). Provides semantic search, AST-based note editing, and workflow state tracking via stdio or SSE transport.

## Development Commands

```bash
# Install dependencies
npm install

# Build (compiles to dist/, excludes test files)
npm run build

# Run all tests (318 tests across 31 files)
npm test

# Run a single test file
npx vitest run src/domain/errors/domain-errors.test.ts

# Run tests in watch mode
npm run test:watch

# Lint (type-check without emitting)
npm run lint

# Docker (uses pre-built image from ghcr.io)
docker compose up
```

## Architecture

Clean Architecture with four layers:

- **`src/domain/`** — Domain errors, port interfaces (`IFileSystemAdapter`, `IEmbeddingProvider`, `IVectorStore`, `IMarkdownRepository`, `IDiffService`, `ITemplateEngine`), value objects (`SafePath`)
- **`src/use-cases/`** — Business logic: AST parsing/patching, chunking, scoring, retrieval, hybrid search, read-by-heading, bulk-read, frontmatter management, update-file, dry-run edit, create-from-template, workflow state, hints, fuzzy matching, wikilink resolution, vault indexing
- **`src/infrastructure/`** — Adapters: `LocalFileSystemAdapter` (fs/promises), `OllamaEmbeddingProvider` (REST), `TransformersEmbeddingProvider` (local `@huggingface/transformers`), `InMemoryVectorStore` (cosine similarity), `MarkdownFileRepository` (AST + frontmatter from file), `UnifiedDiffService` (unified diff via `diff` package), `RegexTemplateEngine` (`{{key}}` placeholder replacement)
- **`src/presentation/`** — 5 MCP tool bindings (`createMcpServer()`), transport layer (`transport.ts`: stdio/SSE selection, Express SSE app)

Entry point: `src/index.ts` — composition root, reads env vars, wires dependencies, selects transport.

### Key Subsystems

- **AST Parser** (`markdown-pipeline.ts`, `ast-navigation.ts`, `ast-patcher.ts`): unified pipeline (remark-parse + remark-gfm + remark-frontmatter) for surgical markdown patching (append/prepend/replace/delete by heading or block ID); supports `replaceMode: body|section`
- **Fragment Retrieval** (`chunker.ts`, `scoring.ts`, `fragment-retrieval.ts`): heading-aware markdown chunking with TF-IDF + word proximity scoring
- **Semantic Search** (`hybrid-search.ts`, `vault-indexer.ts`): hybrid search combining vector similarity with lexical TF-IDF; background auto-vectorization via chokidar file watcher with debounce; supports optional directory scoping via post-filter
- **Embedding Strategy** (`index.ts`): auto-selects provider — local `TransformersEmbeddingProvider` (zero-setup) or `OllamaEmbeddingProvider` when `OLLAMA_URL` is set and reachable
- **Workflow** (`workflow-state.ts`, `hints.ts`): Petri net state machine (IDLE → EXPLORING → EDITING → REVIEWING); contextual hints appended to all tool responses
- **Fuzzy Matching** (`fuzzy-match.ts`): Levenshtein-based typo resilience for edit operations
- **Transport** (`transport.ts`): dual transport — stdio (default, single client) or SSE over HTTP (multi-client); each SSE connection gets its own McpServer + WorkflowStateMachine while sharing fs/vector/embedder deps
- **Vault Search** (`vault-search.ts`): cross-vault lexical keyword search using FragmentRetriever — no embeddings required; supports optional directory scoping
- **Freeform Editor** (`freeform-editor.ts`): line-range replacement and literal string find/replace as fallback for non-AST content
- **Read by Heading** (`read-by-heading.ts`): AST-based section extraction — reads content under a specific heading (up to next same-or-higher-level heading) to save context window space; returns suggestions/guidance if heading not found
- **Frontmatter Management** (`frontmatter.ts`): safe read/update of YAML frontmatter via AST + `js-yaml` — merge fields without touching markdown body; `InvalidFrontmatterPayloadError` for malformed JSON input
- **Update File** (`update-file.ts`): full content replacement with upsert semantics (create or overwrite)
- **Dry-Run Edit** (`dry-run-edit.ts`): coordinates edit preview vs commit — when `dryRun=true`, returns unified diff via `IDiffService` without writing; when false, writes to disk
- **Bulk Read** (`bulk-read.ts`): reads multiple files/heading-scoped sections concurrently in a single call with per-item fault tolerance — reuses `IFileSystemAdapter` and `ReadByHeadingUseCase`
- **Templating** (`create-from-template.ts`, `regex-template-engine.ts`): creates new notes from template files with `{{variable}}` placeholder injection via `ITemplateEngine`; refuses to overwrite existing destination files (`NoteAlreadyExistsError`)
- **5 MCP Tools**: vault (CRUD + update + create_from_template), edit (AST patching + freeform line_replace/string_replace + frontmatter_set + dryRun diff preview + returnContent), view (fragment retrieval + global_search + semantic_search + outline + read by heading + frontmatter_get + bulk_read + backlinks); `view.outline` supports `directory` scoping

### Security


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wirux/mcp-markdown-vault](https://github.com/wirux/mcp-markdown-vault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
