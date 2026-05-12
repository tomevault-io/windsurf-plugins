---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Relic** is a professional artifact storage service with immutable artifacts. Built with FastAPI (Python), Svelte, and Tailwind CSS.

Key principle: Relics cannot be edited after creation - they are permanent and immutable. To modify content, create a fork which creates an independent copy.

### Tech Stack
- **Infrastructure**: Docker + Nginx (Reverse Proxy)
- **Backend**: FastAPI + SQLAlchemy + MinIO/S3
- **Frontend**: Svelte + Tailwind CSS + Axios
- **Database**: PostgreSQL (prod/dev)
- **Storage**: MinIO (dev), S3 (prod)

## Core Architecture Concepts

### 1. Immutable Relic Model

- Each relic is permanent and cannot be edited after creation
- "Forking" creates an independent copy with `fork_of` reference to the source
- Each relic has a unique URL that serves as the access identifier
- No versioning - each relic stands alone

**Database fields:**
- `id`: 32-character hexadecimal (GitHub Gist-style), primary key
- `fork_of`: Source relic if forked (null for original relics)
- `client_id`: Client identification key (nullable for anonymous)
- `name`: Optional display name
- `description`: Optional description
- `content_type`: MIME type of the stored content
- `language_hint`: Optional language hint for syntax highlighting
- `size_bytes`: Size of the content in bytes
- `s3_key`: Storage location (format: `relics/{id}`)
- `access_level`: public (listed in recents) or private (URL is the access token)
- `password_hash`: Optional password protection
- `created_at`, `expires_at`: Timestamps
- `access_count`: Number of times the relic has been accessed

### 2. Universal Content Support

The system handles **any file type** (text, code, images, PDFs, CSVs, archives, relic indexes, etc.):

- **Text/Code**: Displayed with syntax highlighting support via language hints
- **Images**: Displayed as-is with size information
- **PDFs**: Downloaded and can be viewed in browser
- **CSV/Excel**: Downloaded and can be opened in external tools
- **Videos/Archives**: Downloaded and can be processed locally
- **Relic Indexes (.rix)**: Collections of relics with custom metadata and progressive loading


### 3. Fork Relationships

```
Original:  f47ac10b58cc4372a5670e02b2c3d479
  └─ Fork: a1b2c3d4e5f678901234567890abcdef (fork_of: f47ac...)
      └─ Fork: 1234567890abcdef1234567890abcdef (fork_of: a1b2c...)
```

Key queries:
- Check if fork: look at `fork_of` field (null = original relic)
- Get forks of a relic: query where `fork_of` = relic_id
- Trace fork lineage: follow `fork_of` references backward
- Each fork is independent - changes to original don't affect forks

### 4. Access Control & Expiration

- **Access levels**:
  - **Public**: Listed in recent relics, discoverable via UI
  - **Private**: Not listed in recents, only accessible via direct URL (which serves as the access token with 128 bits of entropy)
- **Optional password protection**: Can be applied to any relic (public or private) for additional security
- **Expiration options**: 1h, 24h, 7d, 30d, never (default: never)
- **Anonymous relics**: No client association
- **URL format**: 32-character hexadecimal (GitHub Gist-style), cryptographically secure, practically collision-proof

## Storage Architecture

- **Primary storage**: S3-compatible (MinIO) - one object per relic
- **Database**: Stores metadata (id, client_id, fork_of, content_type, language_hint, size_bytes, s3_key, created_at, expires_at, access_count, tags, etc.)
- **Max upload**: 100MB (configurable)
- **No S3 versioning needed**: Immutable model means each relic is independent

## API Structure

### Key Endpoint Patterns

All API endpoints are prefixed with `/api/v1` and served via Nginx at `http://localhost`.

```
POST   /api/v1/relics                  Create relic
GET    /api/v1/relics/:id              Get relic metadata
PUT    /api/v1/relics/:id              Update relic metadata
GET    /:id/raw                        Get raw content (served from root)
POST   /api/v1/relics/:id/fork         Create fork (independent copy)
DELETE /api/v1/relics/:id              Delete relic (owner OR admin)

GET    /api/v1/relics                  List recent public relics
```

### Admin Endpoints

Admin endpoints require the `X-Client-Key` header with an admin client ID (configured via `ADMIN_CLIENT_IDS` env var).

```
GET    /api/v1/admin/check             Check admin status (no auth required)
GET    /api/v1/admin/relics            List all relics (including private)
GET    /api/v1/admin/clients           List all clients
GET    /api/v1/admin/stats             Get system statistics
DELETE /api/v1/admin/clients/:id       Delete a client
```

**Admin Privileges:**
- Delete any relic (not just their own)
- View all relics including private ones via admin endpoints
- View all registered clients
- Delete clients (and optionally their relics)
- View system statistics

**Setting Up Admin Users:**
1. Get client ID from browser: `localStorage.getItem('relic_client_key')`
2. Add to `ADMIN_CLIENT_IDS` in `docker-compose.prod.yml` (comma-separated for multiple admins)
3. Restart services with `make down && make up`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ovidiuvio/relic](https://github.com/ovidiuvio/relic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
