---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PowerX is a **modular, extensible, pluggable business engine**. It's not a traditional monolithic application but rather a "business kernel + plugin marketplace" architecture. The system enables CRM, e-commerce, SCRM, and other business modules to coexist as plugins on a unified foundation.

**Core Philosophy:**
- **Minimal Kernel**: Core provides only universal capabilities (IAM, RBAC, Event Bus, Audit, DB Layer, Flow Engine)
- **Plugin-First**: All business functionality lives as plugins with independent deployment and database schemas
- **Contract-Driven**: Plugins communicate via contracts/interfaces and event topics, not direct dependencies

## Repository Structure

### Backend (Go)
```
/private/var/www/html/ArtisanCloud/X/PowerX/Core/PowerX/backend/
├── cmd/                    # Binaries (agent/, demo/, test_agent_api/)
├── api/                    # HTTP/gRPC endpoints and handlers
├── internal/               # Application modules (not for external reuse)
│   ├── app/                # Application layer
│   ├── bootstrap/          # Bootstrap configuration
│   ├── contract/           # Contract definitions
│   ├── infra/              # Infrastructure layer
│   │   ├── database/       # Database abstractions (GORM)
│   │   ├── auth/           # Authentication/Authorization
│   │   ├── plugin/         # Plugin system
│   │   ├── cache/          # Caching (Redis)
│   │   ├── media/          # File storage (S3/MinIO)
│   │   └── transport/      # HTTP/gRPC/WebSocket transport
│   └── transport/          # Transport layer
├── pkg/                    # Reusable libraries
│   ├── corex/              # Core SDK capabilities
│   │   ├── rbac/           # RBAC implementation
│   │   └── rls/            # Row-level security
│   ├── auth/               # Auth utilities
│   ├── plugin_mgr/         # Plugin management
│   ├── comm/               # WebSocket/SSE communication
│   └── utils/              # Utilities
├── domain/                 # Domain models and logic
├── config/                 # Configuration loading/validation
├── storage/                # Storage interfaces
├── tests/                  # Test suites
├── specs/                  # Specification documents
└── plugins/, extensions/   # Optional plugin implementations
```

### Frontend (Nuxt)
```
/private/var/www/html/ArtisanCloud/X/PowerX/Core/PowerX/web-admin/
├── app/                    # Nuxt pages and components
├── public/                 # Static assets
├── nuxt.config.ts          # Nuxt configuration
├── package.json            # Dependencies (Nuxt 4, TypeScript, ECharts, Pinia)
└── .env                    # Environment variables
```

## Common Commands

### Backend Development
```bash
# Development
make dev                    # Run demo server (port 8077, logs to stdout)
make dev-agent              # Run agent service
make dev-watch              # Hot reload (requires 'air' tool)

# Build
make build                  # Build all binaries to bin/
make build-agent            # Build agent service
make build-demo             # Build demo service
make build-cross            # Cross-compile for Linux/Windows/macOS
make build-release          # Release build with version tags

# Testing
make unit-test              # Run Go unit tests (go test ./...)
make unit-test-eino         # Run Eino agent tests specifically
make test-all               # Full API test suite (requires running server)
make test-health            # Health check test
make test-quick             # Quick health + config tests
make test-coverage          # Generate coverage report (reports/coverage.html)

# Database
make db-migrate             # Run database migrations
make db-rollback            # Rollback migrations
make db-seed                # Seed database with initial data
make db-refresh             # Rollback + migrate + seed
make db-status              # Check migration status

# Code Quality
make format                 # Format code (go fmt)
make vet                    # Static analysis (go vet)
make check-all              # Full lint check (requires golangci-lint)
make generate               # Run go generate
make docs-api               # Generate Swagger docs (requires swag)

# Utilities
make deps-tidy              # Sync go.mod/go.sum
make logs-tail              # Follow application logs
make profile                # Start pprof (visit http://localhost:8077/debug/pprof/)
make install-tools          # Install dev tools (air, golangci-lint, swag)
```

### Frontend Development
```bash
cd web-admin
npm install                 # Install dependencies
npm run dev                 # Start dev server (port 3000)
npm run build               # Build for production
npm run preview             # Preview production build
```

### Environment Variables
```bash
# Backend
DEV_PORT=8077               # Server port (default: 8077)
LOG_LEVEL=debug             # Logging level
PGHOST, PGPORT, PGUSER      # PostgreSQL connection
PGDATABASE=corex            # Database name

# Frontend
NITRO_PORT=3000             # Nuxt dev server port
```

## Key Components

### 1. Plugin System

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArtisanCloud/PowerX](https://github.com/ArtisanCloud/PowerX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
