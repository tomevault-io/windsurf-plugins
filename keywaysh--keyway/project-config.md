---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Keyway is a GitHub-native secrets management platform. If you have repo access, you get secret access.

This is a **monorepo** using pnpm workspaces + Turborepo, containing:

| Package | Path | Language | Framework |
|---------|------|----------|-----------|
| Backend API | `packages/backend/` | TypeScript | Fastify 5, Drizzle ORM |
| Dashboard | `packages/dashboard/` | TypeScript | Next.js 15 |
| Crypto | `packages/crypto/` | Go 1.25 | gRPC |
| CLI | `packages/cli/` | Go 1.25 | Cobra |
| MCP Server | `packages/mcp/` | TypeScript | MCP SDK |
| Docs | `packages/docs/` | TypeScript | Docusaurus 3 |

**Separate repos** (not in this monorepo): `keyway-action` (GitHub Action), `keyway-admin` (private), `keyway-landing` (private).

**Deployment**:
- **Production SaaS**: Vercel (dashboard) + Railway (backend, crypto, db)
- **Self-hosting**: Docker Compose + Caddy (this repo's docker-compose.yml)

## Development Commands

### Root (Turborepo)
```bash
pnpm install          # Install all TS dependencies
pnpm build            # Build all TS packages
pnpm test             # Run all tests
pnpm lint             # Lint all packages
pnpm dev              # Dev servers (all TS packages)
```

### Backend (`packages/backend/`)
```bash
pnpm --filter keyway-api dev          # Dev server with tsx watch
pnpm --filter keyway-api build        # TypeScript build
pnpm --filter keyway-api type-check   # Type checking only
pnpm --filter keyway-api test         # Run tests
pnpm --filter keyway-api db:generate  # Generate Drizzle migrations
pnpm --filter keyway-api db:migrate   # Run migrations
pnpm --filter keyway-api validate     # Pre-push checks
```

### CLI (`packages/cli/`)
```bash
cd packages/cli
make build            # Build binary
make test             # Run tests
make lint             # Run golangci-lint
```

### Dashboard (`packages/dashboard/`)
```bash
pnpm --filter keyway-dashboard dev    # Next.js dev server
pnpm --filter keyway-dashboard build  # Production build
pnpm --filter keyway-dashboard lint   # ESLint
pnpm --filter keyway-dashboard test   # Run tests
```

### Crypto (`packages/crypto/`)
```bash
cd packages/crypto
go run .              # Run dev server
go test ./...         # Run tests
go build .            # Build binary
```

### MCP (`packages/mcp/`)
```bash
pnpm --filter @keywaysh/mcp dev      # Dev server
pnpm --filter @keywaysh/mcp build    # Build (tsup)
pnpm --filter @keywaysh/mcp test     # Run tests
```

### Docs (`packages/docs/`)
```bash
pnpm --filter keyway-docs start      # Dev server at localhost:3000
pnpm --filter keyway-docs build      # Production build
```

### Root Makefile
```bash
make help             # Show all targets
make setup            # First-time setup (secrets, hosts, certs)
make install          # pnpm install + go mod download
make dev              # Start all services (crypto, backend, dashboard)
make dev-backend      # Backend only
make dev-dashboard    # Dashboard only
make dev-crypto       # Crypto only
make build            # Build all (turbo + go)
make test             # Run all tests
make lint             # Lint all packages
make docker           # docker compose up --build
make clean            # Clean build artifacts
```

## Architecture

### Backend (`packages/backend/src/`)
- `index.ts` - Fastify server entry point
- `api/v1/routes/` - Route handlers (auth, vaults, secrets, billing, integrations)
- `services/` - Business logic (secret, vault, usage services)
- `db/schema.ts` - Drizzle ORM schema (users, vaults, secrets tables)
- `utils/encryption.ts` - AES-256-GCM encryption/decryption
- `utils/github.ts` - GitHub API client for repo access checks
- `middleware/auth.ts` - JWT authentication middleware

### CLI (`packages/cli/internal/`)
- `cmd/` - Cobra commands (login, init, push, pull, run, diff, scan, sync)
- `api/` - Keyway API client
- `auth/` - Token storage via keyring
- `git/` - Git repository detection
- `env/` - Env file parsing and diffing
- `ui/` - Terminal UI helpers

### Dashboard (`packages/dashboard/app/`)
- `(dashboard)/` - Authenticated dashboard routes
- `auth/callback/` - OAuth callback handling
- `components/dashboard/` - Vault cards, secret rows, modals

## Key Patterns

- **Authentication flow**: Device code flow (CLI starts, user approves in browser) or fine-grained PAT
- **Authorization**: All vault access verified via GitHub API collaborator checks
- **Encryption**: Secrets encrypted at rest with AES-256-GCM via isolated crypto gRPC service
- **Analytics**: PostHog for usage metrics; never tracks secret values, only metadata
- **Environment detection**: CLI auto-detects GitHub remote from `.git/config`

## Environment Variables

Backend requires: `DATABASE_URL`, `ENCRYPTION_KEY` (32-byte hex), `GITHUB_APP_CLIENT_ID`, `GITHUB_APP_CLIENT_SECRET`, `JWT_SECRET`

Dashboard requires: `NEXT_PUBLIC_KEYWAY_API_URL`, PostHog keys for analytics

CLI can use: `KEYWAY_API_URL` (defaults to production), `KEYWAY_DISABLE_TELEMETRY=1`

Crypto requires: `ENCRYPTION_KEY` (32-byte hex)

## CI/CD

Workflows in `.github/workflows/` with path-based triggers:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keywaysh/keyway](https://github.com/keywaysh/keyway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
