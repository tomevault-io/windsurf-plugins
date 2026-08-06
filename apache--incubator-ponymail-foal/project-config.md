---
trigger: always_on
description: Guidance for AI coding agents working on this repository.
---

# AGENTS.md

Guidance for AI coding agents working on this repository.

## Project Overview

Apache Pony Mail Foal is a mailing list archive system. It consists of:

- **`server/`** — Python async HTTP API (aiohttp) that queries ElasticSearch
- **`webui/`** — Static JavaScript/HTML frontend served by a reverse proxy
- **`tools/`** — Python CLI utilities for archiving, importing, and migrating email

## Repository Layout

```
server/
├── main.py                    # Entry point, request routing, endpoint discovery
├── ponymail.yaml.example      # Sample configuration
├── openapi.yaml               # OpenAPI 3.0 specification
├── server_version.py          # Auto-generated version from git
├── endpoints/                 # API endpoint handlers (one file per endpoint)
│   ├── stats.py               # Search/list emails
│   ├── email.py               # Fetch single email
│   ├── thread.py              # Fetch email thread
│   ├── source.py              # Raw RFC 2822 source
│   ├── mbox.py                # Mbox download
│   ├── compose.py             # Send email (authenticated)
│   ├── preferences.py         # User prefs + list overview
│   ├── mgmt.py                # Admin GDPR operations
│   ├── oauth.py               # OAuth login flow
│   ├── pminfo.py              # Server activity info
│   ├── gravatar.py            # Avatar caching proxy
│   └── plain.py               # SEO-friendly HTML rendering
├── plugins/                   # Shared server internals
│   ├── configuration.py       # YAML config parsing (source of truth for all config keys)
│   ├── database.py            # ElasticSearch async client pool
│   ├── messages.py            # Email query, threading, trimming logic
│   ├── session.py             # Cookie-based sessions + OAuth credential tracking
│   ├── aaa.py                 # Access control (public vs private lists)
│   ├── defuzzer.py            # Date/query parameter normalization
│   ├── background.py          # Periodic index refresh tasks
│   ├── formdata.py            # Request body parsing (form vs JSON)
│   ├── offloader.py           # Thread pool for CPU-bound JSON serialization
│   ├── auditlog.py            # Admin action logging
│   └── oauth*.py              # Provider-specific OAuth token exchange
└── testendpoints/             # Extra endpoints loaded with --testendpoints

tools/
├── archiver.py                # Pipe-based email archiver (called from /etc/aliases)
├── import-mbox.py             # Bulk mbox file importer
├── migrate.py                 # Migration from old PonyMail databases
├── setup.py                   # First-time ES index setup
├── rethread.py                # Re-compute threading for existing emails
├── bulk-edit.py               # Batch metadata edits
├── archiver.yaml              # Archiver configuration
└── plugins/                   # Shared archiver internals
    ├── elastic.py             # Synchronous ES client for tools
    ├── generators.py          # Email ID generation (DKIM-based)
    ├── dkim_id.py             # DKIM permalink generation
    ├── textlib.py             # Text normalization, list-ID handling
    └── mboxo_patch.py         # Mbox format quirks

webui/
├── index.html                 # Front page (list of lists)
├── list.html                  # List view (emails in a list)
├── thread.html                # Thread view
├── admin.html                 # Management console
├── oauth.html                 # OAuth callback handler
├── js/
│   ├── ponymail.js            # Built/concatenated JS (from source/)
│   ├── config.js              # Client-side config (OAuth client IDs, API URL)
│   └── source/                # Individual JS source files (concatenated by build.sh)
│       ├── build.sh           # Concatenation build script
│       ├── search.js          # Search functionality
│       ├── composer.js        # Email compose UI
│       └── ...                # ~24 source files
├── css/
└── images/

test/
├── test_archiver.py           # Unit tests for archiver
├── test_defuzzer.py           # Unit tests for date/query parsing
├── test_msgid.py              # Unit tests for message ID handling
├── itest_integration.py       # Integration tests (requires running ES + server)
└── resources/                 # Test fixtures (sample emails)

docs/
├── index.md                   # Documentation home page
├── architecture.md            # Component diagram, data model, request lifecycle
├── operator_guide.md          # Install & deploy (Docker + production)
├── user_guide.md              # Browse, search, reply
├── admin_guide.md             # Management console, GDPR, bulk ops
├── plugins.md                 # Endpoint & plugin development guide
├── API.md                     # HTTP API endpoint reference
├── api_client_guide.md        # Curl examples, integration patterns
├── search.md                  # Search query syntax
├── configuration.md           # All ponymail.yaml options
├── releases.md                # Versioning & release process
└── STYLEGUIDE.md              # Code style conventions
```

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Server | Python 3.8+, aiohttp, async/await |
| Database | ElasticSearch 7.x+ (async client) |
| Frontend | Vanilla JavaScript (ES2018), jQuery 1.12, Bootstrap |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/incubator-ponymail-foal](https://github.com/apache/incubator-ponymail-foal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
