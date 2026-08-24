---
trigger: always_on
description: Toku (読く — Japanese for "to read") is a private, offline-first personal book manager. It combines the metadata depth of Calibre, the reading tracking of Goodreads, and the analytics of StoryGraph — without any social features. Phases 0–7 are complete (CLI, imports, analytics, web dashboard, native apps, file management, and sync). A watchOS companion app is planned (tracked in #178).
---

# Copilot Instructions for Toku

## Project Overview

Toku (読く — Japanese for "to read") is a private, offline-first personal book manager. It combines the metadata depth of Calibre, the reading tracking of Goodreads, and the analytics of StoryGraph — without any social features. Phases 0–7 are complete (CLI, imports, analytics, web dashboard, native apps, file management, and sync). A watchOS companion app is planned (tracked in #178).

## Non-Negotiable Constraints

Every code contribution, architecture decision, and feature design must uphold these:

1. **Local-first** — The app must work fully offline. All core features (library management, reading tracking, statistics, search, import, export) function without an internet connection. Network access is only for optional metadata enrichment.
2. **No social features** — No friends, followers, feeds, book clubs, or shared reviews. This is a personal tool. The user is the sole audience for their data.
3. **User data ownership** — The user owns 100% of their data in portable, open formats. Full export at any time. No vendor lock-in.
4. **Import as a superpower** — Importing from Goodreads, Calibre, and StoryGraph must be frictionless, idempotent, and lossless. Import quality is a first-impression feature.
5. **CLI-first** — The CLI is the primary interface and a first-class product. Web, iOS, macOS, and Windows are future platforms built on the same core library.

## Architecture

Cargo workspace with 12 crates:

- `toku-core/` — Domain models, traits, state machine, statistics engine. Pure Rust, no I/O. Compiles to native, WASM, FFI.
- `toku-db/` — SQLite persistence, schema migrations (refinery), FTS5 full-text search.
- `toku-import/` — Import implementations: Goodreads CSV, Calibre metadata.db, StoryGraph.
- `toku-meta/` — Metadata fetching: Open Library API (primary), Google Books (fallback). Cover image downloading.
- `toku-files/` — Ebook file management: association, SHA-256 integrity verification, disk organization, and format conversion (via Calibre's `ebook-convert`).
- `toku-cli/` — CLI binary (clap v4). The main entry point.
- `toku-export/` — Export implementations: CSV, JSON, Markdown, BibTeX, canonical backup.
- `toku-ffi/` — C FFI bindings for Swift/Kotlin via `cbindgen`. Used by macOS and iOS apps.
- `toku-web/` — Axum + maud web server. Library views, statistics dashboard, import wizard, OPDS catalog. Started via `toku serve`.
- `toku-desktop/` — Tauri v2 Windows desktop app wrapping the web UI.
- `toku-sync/` — Self-hostable, zero-knowledge sync relay server (stores only client-encrypted ciphertext).
- `toku-sync-client/` — Client-side sync engine: end-to-end encryption, HLC-based conflict resolution, and op propagation.

### Data Boundary Rule

| Data Type | Storage | Network? |
|---|---|---|
| User's book library, reading sessions, notes, ratings | Local SQLite — plaintext at rest (OS full-disk encryption only; app-level DB encryption is future #204) | Never sent unless sync opted in |
| Book metadata from APIs | Cached locally after fetch | Open Library / Google Books (optional) |
| Cover images | Local filesystem, content-addressed (SHA-256) | Fetched once, stored locally forever |
| Import source data (Goodreads CSV, Calibre DB) | Parsed on-device, stored in local DB | Never leaves device |
| Statistics and analytics | Computed locally from user data | Never sent anywhere |

**Where encryption actually applies** — keep these four guarantees distinct; do not conflate them:

- **Local, at rest** — the working `toku.db` is **plaintext**. `Database::open` sets only `journal_mode=WAL` + `foreign_keys=ON`, and `rusqlite` uses bundled SQLite (not SQLCipher), so protection depends entirely on **OS full-disk encryption**. Optional app-level at-rest encryption is future work (#204).
- **In transit (self-host / sync)** — network confidentiality relies on **operator-provided TLS**; Toku does not ship its own transport layer.
- **Relay (zero-knowledge E2E)** — the sync relay stores **only client-encrypted ciphertext** and never sees plaintext or keys.
- **Web dashboard** — `toku serve` is a **trusted local server** that holds plaintext while running; treat it as inside your trust boundary.

### Key Data Model Decisions

- **Book = Edition** for MVP. A nullable `work_id` column enables Work grouping in Phase 3.
- **Ratings**: 0–10 integer, displayed as 5★ with half-star increments. Goodreads-compatible.
- **Identifiers**: ISBN-10, ISBN-13, ASIN, Open Library ID, Goodreads ID. Books without ISBNs are fully supported.
- **Contributors**: Author, Editor, Translator, Illustrator, Narrator (audiobooks) — via BookAuthor join table with role enum.
- **Provenance**: Every metadata field tracks its source (user entry, Goodreads import, Open Library API) and timestamp. User edits always take precedence.

## Conventions

- **License**: MIT — all contributions must be compatible

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kafkade/toku](https://github.com/kafkade/toku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
