---
trigger: always_on
description: This file provides guidance for AI coding agents working on the Alby Hub repository.
---

# AGENTS.md

This file provides guidance for AI coding agents working on the Alby Hub repository.

## Project Overview

Alby Hub is a self-custodial **Nostr Wallet Connect (NWC)** service that bridges Lightning Network wallets with applications supporting the NIP-47 protocol. It supports multiple Lightning backends (LDK, LND, Phoenixd, Cashu) and runs as either a web server or a desktop app (via Wails).

## Tech Stack

- **Backend:** Go 1.25, Echo v4, GORM, SQLite (default) / PostgreSQL
- **Frontend:** React 18, TypeScript, Vite, Tailwind CSS 4, shadcn/ui, Radix UI, Zustand, SWR
- **Desktop:** Wails v2 (produces native desktop app using Go + web frontend)
- **Lightning:** LDK (embedded), LND (gRPC), Phoenixd, Cashu
- **Protocol:** Nostr NIP-47 (Nostr Wallet Connect)

## Project Structure

```text
hub/
├── api/                  # HTTP API handlers and request/response models
├── alby/                 # Alby account integration (OAuth, backups)
├── apps/                 # App connection management
├── cmd/http/main.go      # HTTP server entry point
├── config/               # Configuration management
├── db/                   # Database layer, migrations, queries
├── events/               # Event pub/sub system
├── frontend/             # React frontend (see below)
├── http/                 # HTTP service router
├── lnclient/             # LN abstraction interface + implementations
│   ├── ldk/              # Embedded LDK node
│   ├── lnd/              # LND gRPC client
│   ├── phoenixd/         # Phoenixd client
│   └── cashu/            # Cashu client
├── nip47/                # NIP-47 protocol implementation
│   ├── controllers/      # Per-method request handlers
│   ├── permissions/      # Permission validation
│   └── cipher/           # NIP-04 encryption
├── service/              # Core service orchestration
├── swaps/                # Boltz atomic swap integration
├── transactions/         # Transaction tracking and metadata
├── tests/                # Test helpers and utilities
└── wails/                # Wails desktop-specific handlers
```

### Frontend Structure

```text
frontend/src/
├── components/           # Reusable UI components
├── screens/              # Page-level route components
├── contexts/             # React context providers
├── hooks/                # Custom React hooks
├── state/                # Zustand client state stores
├── lib/                  # Auth, backend type helpers
├── utils/                # Shared utilities (request.ts, swr.ts, formatting, etc.)
├── types.ts              # Shared TypeScript types
└── routes.tsx            # Route definitions

frontend/platform_specific/
├── http/                 # Web-specific utilities (copied at build time)
└── wails/                # Desktop-specific utilities (copied at build time)
```

## Development Setup

### Prerequisites

- Node.js 20+
- Yarn

### Running in HTTP Mode (Primary)

```bash
# Terminal 1 – Frontend (port 5173)
cd frontend
yarn install
yarn dev:http

# Terminal 2 – Backend (port 8080)
cp .env.example .env   # configure as needed
go run cmd/http/main.go
```

### Running in Desktop Mode (Wails)

```bash
wails dev -tags "wails"
```

## Testing

### Go Backend

```bash
# Run all tests
go test ./...

# Run specific test by name
go test ./... -run TestHandleGetInfoEvent

# Run with PostgreSQL (optional)
export TEST_DATABASE_URI="postgresql://user:password@localhost:5432/postgres"
go test ./...
```

Mocks are generated with `mockery` (config in `.mockery.yaml`); run it after changing any interface.

### Frontend

```bash
cd frontend
yarn lint          # ESLint + TypeScript type check + Prettier
yarn tsc:compile   # TypeScript only
yarn format        # Prettier only
```

No Jest/Vitest tests exist; frontend quality is enforced via linting.

## Building

```bash
# HTTP production build
cd frontend && yarn build:http
go build -o main cmd/http/main.go

# Docker
docker build . -t albyhub:latest
```

## Key Architecture Patterns

### Request Flow

```text
HTTP Request / NIP-47 Nostr Event
    → HTTP Handler / NIP-47 Event Handler
    → api/ package (business logic)
    → LNClient interface
    → Backend implementation (LDK/LND/Phoenixd/Cashu)
```

### Event System

Services communicate via `events/` pub/sub. Prefer publishing events over direct inter-service calls. Key events use the `nwc_*` prefix (e.g., `nwc_payment_sent`, `nwc_payment_received`).

### Platform-Specific Frontend Code

Code under `frontend/platform_specific/http/` and `frontend/platform_specific/wails/` is swapped at build time. Any platform-specific frontend logic must have both variants.

### LNClient Interface

`lnclient/models.go` defines the interface all backends must implement. Changes to this interface require updates to all four implementations (LDK, LND, Phoenixd, Cashu) and their mocks.

## Database

- **Migrations:** `db/migrations/` — always add new migrations here; never modify existing ones.
- **SQLite:** WAL mode, 5s busy timeout, 20MB cache. Default for development and most deployments.
- **PostgreSQL:** Supported for production. If touching DB code, test with both.
- **ORM:** GORM with `go-gormigrate`. Use GORM conventions for new models.

## Coding Conventions

### Go


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getAlby/hub](https://github.com/getAlby/hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
