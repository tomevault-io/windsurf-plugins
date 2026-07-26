---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RequestRepo is an HTTP/DNS/SMTP request analysis tool for security researchers and developers. It captures and displays incoming requests in real-time, allowing users to:
- Inspect HTTP requests with full headers, body, and metadata
- Monitor DNS queries to custom subdomains
- Capture SMTP emails sent to the domain
- Define custom HTTP responses with headers/body
- Set custom DNS records (A, AAAA, CNAME, TXT)
- Share individual requests via secure tokens

## Tech Stack

| Component | Technology |
|-----------|------------|
| Backend | Rust (Tokio async runtime) |
| Frontend | React 19 + Vite + TypeScript |
| UI Library | HeroUI (Tailwind-based) |
| State | Zustand |
| Data Fetching | TanStack React Query |
| Editor | Monaco Editor |
| Build | Cargo (Rust) + Bun (Frontend) |
| Container | Docker (multi-stage alpine) |

## Development Commands

```bash
# Installation
make install           # Install all dependencies and git hooks
make install-deps      # Install Cargo + Bun dependencies only

# Development
make start-backend     # Start Rust backend (HTTP/HTTPS/DNS/SMTP)
make dev-backend       # Start with hot reload (cargo watch)
make start-frontend    # Start Vite dev server

# Building
make build             # Build Rust backend in release mode

# Testing
make test              # Run all tests
make test-backend      # Rust tests only (cargo test)
make test-frontend     # Frontend tests only

# Code Quality
make lint              # Run clippy + eslint
make format            # Run cargo fmt + prettier
make lint-rust         # Rust only
make lint-js           # JavaScript only

# Docker
make docker-build      # Build Docker image
make docker-up         # Start with docker-compose
make docker-down       # Stop containers
```

## Architecture

```
requestrepo/
├── src/                    # Rust backend
│   └── src/
│       ├── main.rs         # Entry point, server startup
│       ├── lib.rs          # Library exports
│       ├── cache/          # In-memory LRU cache (compressed)
│       ├── certs/          # TLS/ACME certificate management
│       │   ├── acme.rs     # Let's Encrypt DNS-01 challenge
│       │   ├── tls.rs      # TLS configuration
│       │   └── storage.rs  # Certificate persistence
│       ├── dns/            # DNS server (UDP/TCP)
│       ├── http/           # HTTP server
│       │   ├── routes.rs   # Legacy v1 routes
│       │   ├── routes_v2.rs # API v2 endpoints
│       │   └── websocket.rs # Real-time updates
│       ├── smtp/           # SMTP server
│       ├── tcp/            # Custom TCP port handling
│       ├── ip2country/     # IP geolocation (DB-IP)
│       ├── models/         # Shared data structures
│       ├── utils/          # JWT auth, config
│       └── tests/          # Integration tests
│
├── frontend/               # React SPA
│   └── src/
│       ├── main.tsx        # Entry point
│       ├── App.tsx         # Router setup
│       ├── api/            # API client (axios)
│       ├── components/     # Reusable UI components
│       │   ├── auth/       # Auth overlays
│       │   ├── file-tree/  # Response file editor tree
│       │   ├── layout/     # App shell (Sidebar, Topbar)
│       │   └── ui/         # Generic UI (ContextMenu)
│       ├── features/       # Feature modules
│       │   └── requests/   # Request display components
│       ├── hooks/          # Custom React hooks
│       │   ├── useWebSocket.ts    # WebSocket connection
│       │   ├── useAutoSession.ts  # Auto session creation
│       │   └── useTheme.ts        # Theme switching
│       ├── lib/            # Utilities
│       │   ├── config.ts   # Runtime configuration
│       │   └── fileTree.ts # File tree logic
│       ├── pages/          # Route components
│       │   ├── RequestsPage.tsx
│       │   ├── DnsSettingsPage.tsx
│       │   └── ResponseEditorPage.tsx
│       ├── stores/         # Zustand stores
│       │   ├── authStore.ts
│       │   ├── sessionStore.ts
│       │   ├── requestStore.ts
│       │   ├── themeStore.ts
│       │   └── uiStore.ts
│       └── types/          # TypeScript definitions
│
├── ip2country/             # Geolocation database (optional)
├── Dockerfile              # Multi-stage build
├── docker-compose.yml      # Container orchestration
└── Makefile                # Development commands
```

## Backend Modules

### Cache (`cache/mod.rs`)
- In-memory LRU cache with gzip compression
- Stores sessions, requests, DNS records, files
- No external dependencies (replaces Redis)
- Configurable max entries and TTL

### Certificates (`certs/`)
- Auto-TLS via Let's Encrypt (production) or self-signed (dev)
- DNS-01 challenge for wildcard domain certificates
- HTTP-01 challenge for IP address certificates (short-lived, 6-day)
- SNI-based dual cert selection: domain cert when SNI present, IP cert when absent
- Automatic renewal before expiration (separate cycles for domain/IP)
- Certificate persistence to disk

### DNS Server (`dns/mod.rs`)
- UDP and TCP listeners
- Custom record support per session: A, AAAA, CNAME, TXT
- Wildcard subdomain matching
- All queries logged to session


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adrgs/requestrepo](https://github.com/adrgs/requestrepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
