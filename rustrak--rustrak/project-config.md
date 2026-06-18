---
trigger: always_on
description: > **Context Architecture Note**: This is the **root context** file for the Rustrak project.
---

# Rustrak - Error Tracking System

> **Context Architecture Note**: This is the **root context** file for the Rustrak project.
> For component-specific context, see:
> - Server (Rust backend): `apps/server/CLAUDE.md`
> - WebView UI (Next.js frontend): `apps/webview-ui/CLAUDE.md`
> - Client Package (TypeScript API client): `packages/client/CLAUDE.md`

## Project Vision

Rustrak is an ultra-lightweight, self-hosted error tracking system compatible with Sentry SDKs. The key differentiator is the separation of concerns:

- **Server (Rust/Actix-web)**: API-only backend, minimal memory footprint (~50-100MB)
- **Frontend (Next.js)**: Optional separate dashboard, can be self-hosted or use Vercel

This architecture allows users to deploy just the server for maximum efficiency, connecting it to any Sentry SDK.

## Architecture

```
┌─────────────────┐     ┌─────────────────────┐     ┌─────────────────┐
│   Sentry SDK    │────▶│   Rustrak Server    │────▶│   PostgreSQL    │
│   (any app)     │     │   (Rust/Actix-web)  │     │                 │
└─────────────────┘     └─────────────────────┘     └─────────────────┘
                                  │
                                  ▼
                        ┌─────────────────┐
                        │  Rustrak UI     │ (optional)
                        │  (Next.js)      │
                        └─────────────────┘
```

## Tech Stack

### Server (`apps/server`)
- **Language**: Rust (2021 edition)
- **Framework**: Actix-web 4.x
- **Database**: PostgreSQL 16 with SQLx
- **Async Runtime**: Tokio
- **Background Processing**: Tokio tasks (in-process)

### Client Package (`packages/client`)
- **Language**: TypeScript 5.9+ (strict mode)
- **HTTP Client**: ky 1.14+ (~3KB)
- **Validation**: Zod 4+ (~10KB, runtime type safety)
- **Build**: tsup (esbuild-based)
- **Testing**: Vitest + MSW (97% coverage)

### Test Sentry Package (`packages/test-sentry`)
- **Language**: TypeScript 5.9
- **Purpose**: CLI tool to send test errors to Sentry-compatible endpoints
- **Usage**: `pnpm test-sentry --dsn <dsn> --all` or individual error types
- **Supports**: errors, warnings, logMessages, breadcrumbs, contexts, user, tags

### WebView UI (`apps/webview-ui`)
- **Framework**: Next.js 16.1 (App Router)
- **Language**: TypeScript 5.9
- **Styling**: Tailwind CSS 4.1
- **UI**: Radix UI + shadcn/ui patterns

## Directory Structure

```
rustrak/
├── CLAUDE.md              # This file - project culture
├── apps/
│   ├── server/            # Rust API server
│   │   ├── CLAUDE.md      # Server-specific context
│   │   ├── Cargo.toml
│   │   ├── Dockerfile
│   │   └── src/
│   └── webview-ui/        # Next.js dashboard
│       ├── CLAUDE.md      # UI-specific context
│       └── src/
├── packages/
│   └── client/            # TypeScript API client
│       ├── CLAUDE.md      # Client-specific context
│       ├── src/           # Client source code
│       ├── tests/         # Vitest + MSW tests (133 tests, 97% coverage)
│       └── dist/          # Build output (ESM + CJS + DTS)
├── docs/
│   ├── ingestion-flow.md  # Event ingestion documentation
│   ├── api-design.md      # API specification
│   ├── database-schema.md # DB schema documentation
│   └── ...
├── docker-compose.yml     # Local development setup
├── turbo.json             # Turborepo config
└── package.json           # Workspace root (pnpm)
```

## Key Concepts

### Sentry Compatibility
Rustrak accepts events from any Sentry SDK using the standard Sentry envelope protocol. The DSN format is:
```
http://<sentry_key>@<host>/<project_id>
```

### Two-Phase Ingestion
1. **Ingest (synchronous, <50ms)**: Parse, validate, store temporarily, return 200
2. **Digest (asynchronous)**: Calculate grouping, create/update issues, store event

### Issue Grouping
Events are grouped into Issues using a deterministic algorithm based on:
- Custom fingerprint (if provided by SDK)
- Exception type + first line of message + transaction
- Log message + transaction
- Fallback grouping

### Authentication Architecture

Rustrak uses **two separate authentication methods** for different use cases:

#### 1. Session Authentication (Web UI)
- **Method**: Email + password login with httpOnly cookies
- **Use case**: Human users accessing the web dashboard
- **Implementation**:
  - Argon2id password hashing (OWASP recommended)
  - actix-session middleware for session management
  - Server Components verify auth via `getCurrentUser()` helper
  - Client Components only for interactive forms
- **Flow**:
  1. User logs in at `/auth/login`
  2. Server creates httpOnly session cookie
  3. All subsequent requests include cookie automatically
  4. Server validates session on each request

#### 2. Token Authentication (SDK/API)
- **Method**: Bearer tokens (40-char hex strings)
- **Use case**: Sentry SDKs sending events, programmatic API access
- **Implementation**:
  - Tokens managed via `/settings/tokens` in web UI
  - Validated via `BearerAuth` extractor on API endpoints
  - SentryAuth extractor for SDK ingestion endpoints
- **Flow**:
  1. Create API token in web UI
  2. Include in SDK DSN or API request headers
  3. Server validates token against `auth_tokens` table

**Why Two Methods?**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustrak/rustrak](https://github.com/rustrak/rustrak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
