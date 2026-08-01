---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

**stegodon** is an SSH-first fediverse multi-user blog written in Go using [Charm Tools](https://github.com/charmbracelet). Users connect via SSH and create notes in a terminal interface. Notes can be subscribed to via RSS and federate via ActivityPub to the Fediverse (Mastodon, Pleroma, etc.) and optionally viewed in a web browser.

## Build and Run Commands

```bash
# Build and run
go build && ./stegodon

# Run tests
go test ./...

# Development workflow (always run after changes)
go clean && go test ./... && go build

# Run with ActivityPub enabled
STEGODON_WITH_AP=true STEGODON_SSLDOMAIN=yourdomain.com ./stegodon

# Run in single-user mode
STEGODON_SINGLE=true ./stegodon

# Run with closed registration
STEGODON_CLOSED=true ./stegodon

# Run in SSH-only mode (no web UI, but RSS/ActivityPub still work)
STEGODON_SSH_ONLY=true ./stegodon
```

## Configuration

Environment variables:
- `STEGODON_HOST` - Server IP (default: 127.0.0.1)
- `STEGODON_SSHPORT` - SSH port (default: 23232)
- `STEGODON_HTTPPORT` - HTTP port (default: 9999)
- `STEGODON_SSLDOMAIN` - Public domain for ActivityPub (default: example.com)
- `STEGODON_WITH_AP` - Enable ActivityPub (default: false)
- `STEGODON_SINGLE` - Single-user mode (default: false)
- `STEGODON_CLOSED` - Close registration (default: false)
- `STEGODON_SSH_ONLY` - SSH-only mode, disables web UI but keeps RSS/ActivityPub (default: false)
- `STEGODON_NODE_DESCRIPTION` - NodeInfo description
- `STEGODON_MAX_CHARS` - Maximum note length (default: 150, max: 300)
- `STEGODON_SHOW_GLOBAL` - Show global timeline in TUI and web (default: false)
- `STEGODON_SHOW_TOS` - Show terms of service acceptance screen (default: false)
- `STEGODON_IDLE_TIMEOUT` - SSH session idle timeout in minutes (default: 30)
- `STEGODON_WITH_JOURNALD` - Linux journald logging (default: false)
- `STEGODON_WITH_PPROF` - Enable pprof on localhost:6060 (default: false)

File locations:
- Config: `~/.config/stegodon/config.yaml` (or `./config.yaml`)
- Database: `~/.config/stegodon/database.db` (or `./database.db`)
- SSH key: `~/.config/stegodon/.ssh/stegodonhostkey`

## Architecture

### Application Lifecycle

The application uses a structured lifecycle pattern in `app/app.go`:
- `App` struct encapsulates config, SSH server, and HTTP server
- `New()` creates the app instance
- `Initialize()` runs migrations and sets up servers
- `Start()` starts servers and blocks until shutdown signal
- `Shutdown()` gracefully stops HTTP then SSH with 30s timeout

### Dual Server Model

The application runs two concurrent servers:
- **SSH Server** (port 23232): TUI client connections via [wish](https://github.com/charmbracelet/wish)
- **HTTP Server** (port 9999): RSS feeds, web UI, and ActivityPub endpoints via [gin](https://github.com/gin-gonic/gin)

Both servers support graceful shutdown on SIGTERM/SIGINT.

### TUI Architecture

Built with [bubbletea v2](https://charm.land/bubbletea/v2) MVC pattern. Main orchestrator in `ui/supertui.go`.

**Views:**
- `createuser` - First-time username selection
- `writenote` - Note creation (with reply mode)
- `myposts` - User's own notes with edit/delete
- `hometimeline` - Combined local + federated timeline
- `globalposts` - Global timeline (all local + all federated posts, admin-enabled)
- `threadview` - Thread/conversation view
- `profileview` - User profile with avatar and recent posts (supports both local and remote/federated users)
- `followuser` - Follow remote users
- `followers` / `following` - Relationship lists
- `localusers` - Browse local users
- `notifications` - View and manage notifications
- `relay` - Manage ActivityPub relay subscriptions (admin)
- `admin` - Admin panel
- `search` - Full-text search overlay (FTS5, activated with `/` from timelines)
- `accountsettings` - Profile editing, avatar upload, account deletion
- `terms` - Terms of service acceptance screen

**Navigation:** Tab cycles forward, Shift+Tab backward. Press Ctrl+N for notifications. Enter opens threads, Esc returns. Press `/` to search from any timeline view.

### Database Layer

SQLite with WAL mode. Singleton pattern with connection pooling (max 25 connections).

**Core tables:** `accounts`, `notes`, `hashtags`, `note_hashtags`

**ActivityPub tables:** `follows`, `remote_accounts`, `activities`, `likes`, `boosts`, `delivery_queue`, `note_mentions`, `relays`

**Admin/feature tables:** `bans`, `notifications`, `info_boxes`, `server_message`, `upload_tokens`, `terms_and_conditions`, `user_terms_acceptance`

**Full-text search tables:** `posts_fts` (FTS5 virtual table, content + author only), `posts_fts_lookup` (source_id → fts_rowid mapping with source_type and created_at)

**Denormalized counters:** `reply_count`, `like_count`, `boost_count` on notes and activities for performance.

### ActivityPub Layer

Located in `activitypub/`:
- `httpsig.go` - RSA-SHA256 HTTP signature signing/verification
- `actors.go` - Remote actor fetching and caching (24h TTL)
- `inbox.go` - Incoming activity processing
- `outbox.go` - Outgoing activity sending
- `delivery.go` - Background queue worker with exponential backoff

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deemkeen/stegodon](https://github.com/deemkeen/stegodon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
