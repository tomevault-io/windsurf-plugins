---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Last verified: 2026-06-12 (SyncModel + Settings IA: `internal/source` gained `SyncModel`/`SyncModelFor` — a typed per-source-type sync-semantics descriptor surfaced as `sync_model` on `GET /api/sources` and as Unicode-glyph banners (⇅/⬇) on the Files tabs; Settings split into four deep-linkable groups `/settings/{devices,ai,integrations,system}` (legacy `/settings` 303s to devices), with the Devices group rendering uniform per-source sections. Prior: ForestNote sync device management — /sync/v1 optional `device_name` envelope field; Settings "Sync Devices" card + /api/v1/sync/{devices,compact}; prune = cleanup-only delete of the sync_cursors row, spec §4.3)

Platform-neutral note management and task synchronization service supporting Supernote (via Supernote Private Cloud) and Onyx Boox devices. Six subsystems:
1. **CalDAV task sync** -- CalDAV VTODO over local SQLite task store
2. **Device sync** -- UB *is* the device-facing Supernote Private Cloud server (`internal/spcserver`); Supernote devices connect to UB directly. (The legacy SPC *client* — `internal/tasksync`, `internal/sync`, `internal/db`, MariaDB catalog write-through — was removed 2026-05-25.)
3. **Supernote notes pipeline** -- scans Supernote .note files, extracts/OCRs text, indexes for full-text search
4. **Boox notes pipeline** -- receives Boox .note files via WebDAV, parses ZIP+protobuf, renders strokes, OCRs, indexes for unified search
5. **RAG retrieval pipeline** -- Ollama embeddings, hybrid FTS5+vector search, vLLM-powered chat with retrieval-augmented context
6. **MCP server** -- Model Context Protocol server exposing note search/retrieval tools for AI agents

## Bash Commands: No `cd &&` Compounds

**NEVER** use `cd /path && command` compound bash statements. This triggers a Claude Code bug where the permission prompt fires on `cd` instead of the actual command.

Instead: `git -C /path`, `go -C /path build`, or absolute paths.

## Project Structure

### Core Components
- `cmd/ultrabridge/` -- entry point, wires all components
- `cmd/ub-mcp/` -- MCP server binary: exposes search_notes, get_note_pages, get_note_image tools via stdio or HTTP SSE (see domain CLAUDE.md)

### Configuration & Data Management
- `internal/appconfig/` -- SQLite-backed application config with two-stage loading (bootstrap env vars + settings table), restart detection (see domain CLAUDE.md)
- `internal/notedb/` -- SQLite DB opener + schema migrations for notes, settings, and sources tables (see domain CLAUDE.md)
- `internal/source/` -- Platform-neutral source abstraction: `Source` interface, `SourceRow` model, CRUD operations (see domain CLAUDE.md)
- `internal/source/supernote/` -- Supernote source adapter: .note pipeline, Processor creation (see domain CLAUDE.md)
- `internal/source/boox/` -- Boox source adapter: WebDAV receiver, Processor creation (see domain CLAUDE.md)

### Task Synchronization
- `internal/caldav/` -- CalDAV backend (go-webdav), VTODO conversion with iCal blob overlay (see domain CLAUDE.md)
- `internal/taskstore/` -- Task model, field mapping helpers, MariaDB CRUD (legacy), ErrNotFound sentinel (see domain CLAUDE.md)
- `internal/taskdb/` -- SQLite task store: Open/migrate DB, implements caldav.TaskStore (see domain CLAUDE.md)

### Note Processing & Pipelines
- `internal/processor/` -- background OCR job queue: backup, extract, render, OCR, inject, index (see domain CLAUDE.md)
- `internal/search/` -- FTS5 full-text search over note content (see domain CLAUDE.md)
- `internal/notestore/` -- file inventory (scan, list, get), content hashing, job transfer against SQLite notes table (see domain CLAUDE.md)
- `internal/pipeline/` -- file detection: fsnotify watcher, reconciler, Engine.IO listener (see domain CLAUDE.md)
- `internal/booxpipeline/` -- Boox processing pipeline: store, worker, processor (parse/render/OCR/index) (see domain CLAUDE.md)

### Boox-Specific
- `internal/booxnote/` -- Boox .note ZIP parser: protobuf pages, nested shape ZIPs, binary point files (see domain CLAUDE.md)
- `internal/booxnote/proto/` -- Generated protobuf code for Boox .note format (NoteInfo, VirtualPage, ShapeInfoProto)
- `internal/booxnote/testutil/` -- Exported test helper: builds synthetic .note ZIP files for tests
- `internal/booxrender/` -- Stroke renderer: pressure-sensitive scribbles, geometric shapes via fogleman/gg (see domain CLAUDE.md)
- `internal/webdav/` -- WebDAV server for Boox file uploads with versioning (see domain CLAUDE.md)
- `internal/pdfrender/` -- PDF page rendering via pdftoppm (poppler-utils) for bulk import pipeline

### RAG & Chat
- `internal/rag/` -- RAG embedding infrastructure: Ollama embedder, embedding store with in-memory cache, hybrid FTS5+vector retriever, backfill (see domain CLAUDE.md)
- `internal/chat/` -- Chat subsystem: session/message store (SQLite), vLLM streaming handler with RAG context injection (see domain CLAUDE.md)

### Service Layer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jdkruzr/ultrabridge](https://github.com/jdkruzr/ultrabridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-22 -->
