---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

Relive is an intelligent photo memory frame system that analyzes photos using AI and displays them on various devices. It consists of:
- **Backend**: Go (Gin + GORM + SQLite) - REST API server
- **Frontend**: Vue 3 + TypeScript + Vite + Element Plus - Web management interface
- **relive-analyzer**: Standalone CLI tool for offline AI batch analysis
- **Devices**: Support for multiple hardware platforms (ESP32, Android, iOS, etc.)

## Development Commands

### Root Level (Makefile)
```bash
# Development
make dev              # Start the local development environment (backend + frontend)
make dev-backend      # Start backend only (available for focused debugging)
make dev-frontend     # Start frontend only (available for focused debugging)

# Production
make build            # Build Docker images
make deploy           # Source deployment from the current checkout
make deploy-image     # Deploy using published images (recommended for normal users)
make prod             # Compatibility alias for deploy-image
make stop             # Stop services for the active compose file
make restart          # Restart services for the active compose file
make logs             # View logs for the active compose file

# Testing & Maintenance
make test             # Run backend tests (cd backend && go test -v ./...)
make clean            # Clean build artifacts
make deps             # Install all dependencies
```

### Backend Commands (backend/Makefile)
```bash
cd backend

make build            # Build binary to bin/relive
make run              # Run with dev config (config.dev.yaml)
make test             # Run all tests
make test-coverage    # Generate coverage report (coverage.html)
make lint             # Run golangci-lint
make fmt              # Format with gofmt and goimports
make clean            # Clean build artifacts
make deps             # Download Go modules
```

### Frontend Commands
```bash
cd frontend

npm run dev           # Start dev server (http://localhost:5173)
npm run build         # Type check and build for production
npm run preview       # Preview production build
```

### relive-analyzer (API Analysis Tool)
```bash
# Build
cd backend
go build -o relive-analyzer ./cmd/relive-analyzer

# Generate sample config
./relive-analyzer gen-config > analyzer.yaml

# Usage (API Mode - requires running server)
./relive-analyzer check -config analyzer.yaml                           # Check server connection
./relive-analyzer analyze -config analyzer.yaml                         # Run batch analysis
./relive-analyzer analyze -config analyzer.yaml -workers 10             # Custom concurrency
./relive-analyzer analyze -config analyzer.yaml -verbose                # Verbose logging
./relive-analyzer version                                               # Show version info

# Config file location: analyzer.yaml.example (repository root sample)
```

## Architecture

### Backend Architecture (Layered)

```
HTTP Request → Handler → Service → Repository → Database
                  ↓          ↓           ↓
            Validation   Business    Data Access
                         Logic       (GORM)
```

**Key Layers**:
- **Handler** (`internal/api/v1/handler/`): HTTP request handling, validation, response formatting
- **Service** (`internal/service/`): Business logic, orchestration
- **Repository** (`internal/repository/`): Database access layer (GORM)
- **Model** (`internal/model/`): Data models and DTOs
- **Provider** (`internal/provider/`): AI provider implementations (Ollama, Qwen, OpenAI, VLLM, Hybrid)

**Important Patterns**:
- Repository pattern with interface definitions for testability
- Service layer handles business logic, not repositories
- Handlers use `model.Response` for unified JSON responses
- Configuration via YAML (config.dev.yaml for development)

### Frontend Architecture

```
src/
├── api/           # Axios HTTP clients (photo.ts, config.ts, etc.)
├── views/         # Page components (Photos/, Dashboard/, etc.)
├── layouts/       # Layout components (MainLayout.vue)
├── router/        # Vue Router configuration
├── stores/        # Pinia state management
├── types/         # TypeScript interfaces
├── utils/         # Utility functions (request.ts for HTTP)
└── assets/        # CSS styles with CSS variables
```

**Key Patterns**:
- Composition API with `<script setup lang="ts">`
- API functions in `api/` modules, not inline in components
- Types defined in `types/` matching backend models
- HTTP client configured in `utils/request.ts` with interceptors

### Database (SQLite with GORM)

- Development: `backend/data/relive.db`
- Auto-migration enabled in dev mode (`auto_migrate: true`)
- Key tables: photos, photo_tags, devices, display_records, app_config, users, cities
- `photo_tags` table stores normalized tags (one row per photo-tag pair), dual-written with `photos.tags` for rollback safety

### AI Provider System

Providers implement the `provider.AIProvider` interface:
```go
type AIProvider interface {
    Analyze(request *AnalyzeRequest) (*AnalyzeResult, error)
    AnalyzeBatch(requests []*AnalyzeRequest) ([]*AnalyzeResult, error)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davidhoo/relive](https://github.com/davidhoo/relive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
