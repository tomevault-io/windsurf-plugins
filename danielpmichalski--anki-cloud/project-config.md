---
trigger: always_on
description: > This file serves as the authoritative reference for AI assistants (and human contributors)
---

# CLAUDE.md — Project Architecture & Decisions

> This file serves as the authoritative reference for AI assistants (and human contributors)
> working on this project. It captures all key product decisions, architecture choices, and
> rationale made during the initial design phase.

---

## 1. Project Vision

A **privacy-first Anki sync server** with a first-class REST API and MCP server,
enabling seamless LLM-to-Anki workflows. User deck data is stored in their own cloud storage
(Google Drive, Dropbox, etc.) — the service acts as stateless infrastructure, never holding
user data.

Self-hosters run everything with `docker compose up`. A separate hosted platform (closed-source,
open-core model) wraps this for users who want convenience over control.

### The core workflow this enables

```
User: "<discusses topic with LLM>; create flashcards from this discussion"
LLM:  "<shows proposed flashcards for review>"
User: "looks good, publish to my Anki"
LLM:  "<calls MCP tool> → cards appear in user's Anki instantly"
```

This must work from any LLM UI — desktop app, web UI, API client — without the user having
Anki open or doing anything manually.

---

## 2. The Problem Being Solved

### Why this doesn't exist yet

- **AnkiWeb** (the official sync service) is **closed source** and has no public API
- **AnkiWeb** has no OAuth2 — third-party services would need to store user passwords (unacceptable)
- **AnkiConnect** (the community API plugin) only works locally when Anki Desktop is open
- No existing solution bridges LLMs → Anki in a seamless, cloud-native, privacy-respecting way

### What is and isn't open source in the Anki ecosystem

| Component             | Open Source | Notes                                 |
|-----------------------|-------------|---------------------------------------|
| Anki Desktop          | ✅ AGPLv3    | `ankitects/anki` on GitHub            |
| AnkiDroid (Android)   | ✅ AGPLv3    | Separate community project            |
| anki-sync-server      | ✅ AGPLv3    | Built into Anki Desktop since v25.09  |
| AnkiWeb (hosted sync) | ❌ Closed    | Proprietary, run by Ankitects Pty Ltd |
| AnkiMobile (iOS)      | ❌ Closed    | Paid app, funds Anki development      |

**Key insight:** The sync *protocol* and *server implementation* are open source. Only the
*hosted service* at ankiweb.net is closed. We run our own sync server — no dependency on
Ankitects ever shipping anything.

### Trademark note

**"Anki" is a registered trademark** of Ankitects Pty Ltd (USPTO #79340880, active May 2025).
Do NOT use "Anki" in the product name. Other apps have already received cease & desist letters.

---

## 3. Core Design Decisions

### 3.1 Authentication — [ADR-0004](docs/decisions/0004-use-oauth2-for-authentication-no-password-storage.md) · [ADR-0005](docs/decisions/0005-use-google-as-the-sole-oauth-provider-mvp.md)

OAuth2 only, no passwords. Google as sole provider for MVP.

### 3.2 Storage — [ADR-0002](docs/decisions/0002-use-user-owned-cloud-storage-for-deck-data.md) · [ADR-0006](docs/decisions/0006-use-google-drive-as-the-primary-storage-backend.md)

Deck data lives in user-owned cloud storage. Google Drive for MVP; Dropbox, S3, OneDrive on roadmap.
A `local` provider is also supported for testing/self-hosting without cloud storage.

### 3.3 Sync Protocol — [ADR-0003](docs/decisions/0003-fork-rust-ankitects-sync-server.md)

Fork of the Rust ankitects sync server with a cloud storage adapter layer.

### 3.4 REST API + MCP Server — [ADR-0007](docs/decisions/0007-mcp-server-wraps-rest-api-not-direct-db.md) · [ADR-0008](docs/decisions/0008-use-hono-on-bun-for-rest-api-and-mcp-server.md)

Hono on Bun. Full CRUD API with OpenAPI spec auto-generated from Zod schemas. MCP server wraps the REST API — no direct DB access.

---

## 4. Technical Architecture

### 4.1 System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        User Devices                         │
│  Anki Desktop / AnkiDroid / AnkiMobile / LLM UI             │
└────────────┬────────────────────────────┬───────────────────┘
             │ Anki Sync Protocol          │ MCP Protocol
             ▼                            ▼
┌────────────────────────────────────────────────────────────┐
│                     Your Service                           │
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  Sync Server │  │  REST API    │  │   MCP Server     │  │
│  │  (Rust)      │  │ (Hono/Bun)   │  │   (wraps REST)   │  │
│  └──────┬───────┘  └──────┬───────┘  └──────────────────┘  │
│         │                 │                                │
│  ┌──────▼─────────────────▼──────────────────────────────┐ │
│  │              Auth & Storage Adapter Layer             │ │
│  │  Google OAuth (identity) + Google Drive OAuth (storage)     │ │
│  └──────────────────────────┬────────────────────────────┘ │
│                             │                              │
│  ┌────────────────┐  ┌────────▼──────┐                     │
│  │  SQLite        │  │  Redis        │                     │
│  │  (user table,  │  │  (sessions,   │                     │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielpmichalski/anki-cloud](https://github.com/danielpmichalski/anki-cloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
