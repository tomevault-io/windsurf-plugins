---
trigger: always_on
description: Handles TOON, JSON, JSON-compact, JSONL, and CSV output with field filtering:
---

# CLAUDE.md - workspace-cli Development Guide

## Project Overview

**workspace-cli** is a high-performance Rust CLI for Google Workspace APIs, optimized for AI agent integration. It provides structured output (TOON/JSON/JSONL/CSV) for Gmail, Drive, Calendar, Docs, Sheets, Slides, Tasks, Chat, Contacts, and Groups.

**Author:** Majid Manzarpour
**License:** MIT
**Rust Edition:** 2021

## Quick Command Reference

### Build & Test
```bash
cargo build --release          # Release build (optimized, ~4MB binary)
cargo build                    # Debug build
cargo test                     # Run all tests (9 tests)
cargo clippy                   # Lint check
```

### Binary Location
```
./target/release/workspace-cli   # Release binary
./target/debug/workspace-cli     # Debug binary
```

### Authentication
```bash
workspace-cli auth login --credentials /path/to/credentials.json
workspace-cli auth status
workspace-cli auth logout
```

## Architecture

```
src/
├── main.rs              # CLI entry point, clap command definitions (~3000 lines)
├── lib.rs               # Library exports
├── cli.rs               # CLI context utilities
├── auth/                # OAuth2 & token management
│   ├── oauth.rs         # WorkspaceAuthenticator, SCOPES
│   ├── token.rs         # TokenManager (get_access_token, ensure_authenticated)
│   └── keyring_storage.rs  # OS keyring integration
├── client/              # API client infrastructure
│   ├── api_client.rs    # ApiClient with rate limiting & retry
│   ├── rate_limiter.rs  # Per-API rate limiters
│   ├── retry.rs         # Exponential backoff retry logic
│   └── batch.rs         # BatchClient for multipart/mixed batch requests
├── commands/            # Service implementations
│   ├── gmail/           # list, get, send, reply, delete, trash, labels
│   ├── drive/           # list, upload, download, mkdir, share, etc.
│   ├── calendar/        # list, create, update, delete events
│   ├── docs/            # get, append, create, replace
│   ├── sheets/          # get, update, append, create, clear
│   ├── slides/          # get presentation/page
│   ├── tasks/           # lists, list, create, update, delete
│   ├── chat/            # spaces, messages, read-state, unread, mark-read
│   ├── contacts/        # list, search, get, create, delete, directory
│   ├── groups/          # list, members
│   ├── admin/           # users, reports
│   └── batch/           # CLI wrapper for batch API requests
├── config/              # Config file handling (~/.config/workspace-cli/)
├── error/               # Structured error types (CliError, WorkspaceError)
├── output/              # Formatter (TOON/JSON/JSONL/CSV), field filtering
└── utils/               # base64, field_mask, html_to_md
```

## Key Components

### ApiClient (`src/client/api_client.rs`)
Factory methods create service-specific clients with appropriate rate limiters:
```rust
ApiClient::gmail(token_manager)    // Gmail API client
ApiClient::drive(token_manager)    // Drive API client
ApiClient::calendar(token_manager) // Calendar API client
ApiClient::docs(token_manager)     // Docs API client
ApiClient::sheets(token_manager)   // Sheets API client
ApiClient::slides(token_manager)   // Slides API client
ApiClient::tasks(token_manager)    // Tasks API client
ApiClient::chat(token_manager)     // Chat API client
ApiClient::contacts(token_manager) // Contacts (People) API client
ApiClient::groups(token_manager)   // Groups (Cloud Identity) API client
ApiClient::admin(token_manager)    // Admin Directory API client
```

### API Endpoints (`src/client/api_client.rs:11-19`)
```rust
GMAIL:    "https://gmail.googleapis.com/gmail/v1"
DRIVE:    "https://www.googleapis.com/drive/v3"
CALENDAR: "https://www.googleapis.com/calendar/v3"
DOCS:     "https://docs.googleapis.com/v1"
SHEETS:   "https://sheets.googleapis.com/v4"
SLIDES:   "https://slides.googleapis.com/v1"
TASKS:    "https://tasks.googleapis.com/tasks/v1"
CHAT:     "https://chat.googleapis.com/v1"
CONTACTS: "https://people.googleapis.com/v1"
GROUPS:   "https://cloudidentity.googleapis.com/v1"
ADMIN:    "https://admin.googleapis.com/admin/directory/v1"
```

### Output Formatter (`src/output/formatter.rs`)
Handles TOON, JSON, JSON-compact, JSONL, and CSV output with field filtering:
- `--format toon|json|json-compact|jsonl|csv` (default: toon)
- `--fields "id,name,mimeType"` - Filter output fields
- `--quiet` - Suppress output
- `--output file.json` - Write to file

**Important:** Field filtering handles wrapper objects (`files`, `messages`, `items`, `labels`, `permissions`) by filtering array items, not the wrapper itself.

### Error Handling (`src/error/types.rs`)
Structured errors for agent consumption:
```rust
ErrorCode::AuthenticationFailed
ErrorCode::TokenExpired
ErrorCode::RateLimitExceeded
ErrorCode::QuotaExceeded
ErrorCode::NotFound
ErrorCode::PermissionDenied
ErrorCode::InvalidRequest
ErrorCode::NetworkError
ErrorCode::ServerError
```

## CLI Command Structure

### Global Options (all commands)
```
-f, --format <FORMAT>    Output format: toon, json, jsonl, csv [default: toon]
--fields <FIELDS>        Comma-separated fields to include
-o, --output <FILE>      Write output to file
-q, --quiet              Suppress non-essential output

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [majidmanzarpour/workspace-cli](https://github.com/majidmanzarpour/workspace-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
