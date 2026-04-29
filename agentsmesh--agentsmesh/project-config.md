---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AgentsMesh is **The AI Agent Workforce Platform** — where teams scale beyond headcount. It supports Claude Code, Codex CLI, Gemini CLI, Aider, and more. It consists of four main components:

- **Backend**: Go API server (Gin + GORM)
- **Web**: Next.js frontend (App Router + TypeScript + Tailwind CSS)
- **Web-Admin**: Admin Console frontend (Next.js + Tailwind CSS) - internal management interface
- **Runner**: Go daemon that executes AI agent tasks in isolated PTY environments

## Development Environment (Docker)

**Always use `deploy/dev` Docker environment for development and debugging.** This setup includes Traefik reverse proxy and mirrors production architecture, helping catch issues early.

### Quick Start (Recommended)

```bash
cd deploy/dev
./dev.sh               # One-click: starts Docker backend + local frontend
./dev.sh --clean       # Stop and clean up everything
```

This script automatically:
1. Generates `.env` with worktree-isolated ports (supports multiple worktrees)
2. Starts Docker backend services (PostgreSQL, Redis, MinIO, Backend, Traefik, Runner)
3. Runs database migrations and seeds test data
4. Starts local frontend (Next.js with Turbopack) for better performance

### Services & Ports (main worktree)

| Service | URL | Description |
|---------|-----|-------------|
| **Frontend** | http://localhost:3000 | Next.js (local, Turbopack) |
| **API** | http://localhost:80/api | Backend API via Traefik |
| **Traefik Dashboard** | http://localhost:8080 | Traefik dashboard (dev only) |
| **Adminer** | http://localhost:8081 | Database management UI |
| **MinIO Console** | http://localhost:9001 | S3-compatible storage UI |
| PostgreSQL | localhost:5432 | Database (user: agentsmesh, pass: agentsmesh_dev) |
| Redis | localhost:6379 | Cache |

Test accounts:
- **User**: dev@agentsmesh.local / devpass123
- **Admin**: admin@agentsmesh.local / adminpass123

> **Note**: All ports are dynamically allocated based on worktree. Check `deploy/dev/.env` for actual ports (e.g., WEB_PORT, HTTP_PORT).

### Logs

```bash
tail -f deploy/dev/web.log       # Frontend logs
docker compose logs -f backend   # Backend logs
docker compose logs -f runner    # Runner logs
```

### Hot Reload

- **Frontend**: Next.js Turbopack fast refresh (local)
- **Backend**: Go code auto-rebuild via Air (Docker)
- **Runner**: Go code auto-rebuild (Docker)

## Build Commands (for CI/testing outside Docker)

### Backend (Go)

```bash
cd backend
go build ./cmd/server            # Build binary
go test ./...                    # Run all tests
go test -v ./internal/service/... -run TestAuth  # Run specific test
```

### Web (Next.js)

```bash
cd web
pnpm install                     # Install dependencies
pnpm build                       # Production build
pnpm lint                        # ESLint
pnpm test                        # Run tests (Vitest)
pnpm test:run                    # Run tests once
pnpm test:coverage               # Test coverage
```

### Web-Admin (Next.js)

```bash
cd web-admin
pnpm install                     # Install dependencies
pnpm build                       # Production build
pnpm lint                        # ESLint
pnpm dev                         # Start development server
```

### Runner (Go)

```bash
cd runner
make build                       # Build binary (no CGO required)
make test                        # Run tests
make lint                        # golangci-lint
make build-all                   # Cross-platform builds
```

### Database Migrations

Migrations are located in `backend/migrations/` using golang-migrate format.

**Development** (via Docker):
```bash
cd deploy/dev
./dev.sh               # Automatically runs all migrations
```

**Production** (via backend container):
```bash
# Inside the backend container, golang-migrate is pre-installed
migrate -path /app/migrations -database "postgres://user:pass@host:5432/db?sslmode=disable" up
migrate -path /app/migrations -database "postgres://user:pass@host:5432/db?sslmode=disable" down 1
migrate -path /app/migrations -database "postgres://user:pass@host:5432/db?sslmode=disable" version
```

**Create new migration**:
```bash
# Install golang-migrate locally
brew install golang-migrate

# Create migration files
migrate create -ext sql -dir backend/migrations -seq add_new_feature
# This creates: 000024_add_new_feature.up.sql and 000024_add_new_feature.down.sql
```

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Web (Next.js)                            │
│                 localhost:3000                              │
└─────────────────────────────────────────────────────────────┘
                              │
                        REST / WebSocket
                         (terminal/events)
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   Backend (Go + Gin)                        │
│            REST: localhost:8080 | gRPC: localhost:9443      │
│  - Auth (JWT + OAuth)                                       │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgentsMesh/AgentsMesh](https://github.com/AgentsMesh/AgentsMesh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
