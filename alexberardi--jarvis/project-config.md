---
trigger: always_on
description: Core Jarvis project rules, architecture, and coding standards. Always active.
---


# Jarvis - Personal Voice Assistant

Private, self-hosted voice assistant with Pi Zero nodes and microservices.

**See also:** `CLAUDE.md` and `RULES.md` for full details.

## Core Principles

1. **Fully private and open source** - No cloud dependencies by default, all data stays local
2. **Self-hostable with optional cloud** - Same open-source codebase for both
3. **Fully extensible** - Add capabilities via `IJarvisCommand` interface

## Architecture

```
Pi Zero Nodes → Command Center (7703) → Speech-to-Text (Whisper)
                                      → LLM Processing
                                      → Tool Execution → Service Response
```

## Service Inventory

| Service | Port | Description |
|---------|------|-------------|
| jarvis-auth | 7701 | JWT authentication, user management, app-to-app credentials |
| jarvis-command-center | 7703 | Central voice/command API, node management, tool routing |
| jarvis-whisper-api | 7706 | Speech-to-text via whisper.cpp |
| jarvis-ocr-service | 7031 | OCR with pluggable backends (Tesseract, EasyOCR, Apple Vision) |
| jarvis-recipes-server | 7030 | Recipe CRUD and meal planning |
| jarvis-tts | 7707 | Text-to-speech via Piper TTS |
| jarvis-logs | 7702 | Centralized logging (Loki + Grafana) |
| jarvis-mcp | 7709 | MCP server for Claude Code integration |
| jarvis-config-service | 7700 | Service discovery and configuration |
| jarvis-llm-proxy-api | 7704/7705 | LLM proxy (API + queue worker) |

### Libraries

| Library | Purpose |
|---------|---------|
| jarvis-log-client | Structured logging with async batching |
| jarvis-config-client | Service discovery client |
| jarvis-auth-client | Shared auth (superuser JWT + app-to-app) |
| jarvis-settings-client | Multi-tenant settings with caching |

### Client Software & Apps

| Client | Type | Purpose |
|--------|------|---------|
| jarvis-node-setup | Python (Pi Zero) | Voice node client software |
| jarvis-admin | React 19 + Vite | Web admin dashboard |
| jarvis-installer | React 19 + Vite | Static SPA for install configuration |
| jarvis-node-mobile | React Native + Expo | Mobile companion app for node provisioning |
| jarvis-recipes-mobile | React Native + Expo | Mobile recipes app |

## Data Services (Shared Infrastructure)

Shared infrastructure containers live in `jarvis-data-stores/` (this repo):

| Service | Port | Purpose |
|---------|------|---------|
| PostgreSQL | 5432 | Shared database server (each service has its own DB) |
| Redis | 6379 | Queue for async jobs (OCR, LLM training) |
| MinIO | 9000 | Object storage |
| Mosquitto | 1883 | MQTT broker (node ↔ TTS communication) |

Each backend service connects to the **shared PostgreSQL** but uses its **own database** (e.g., `jarvis_auth`, `jarvis_command_center`, `jarvis_recipes`).

## Common Patterns

- **Framework**: FastAPI + Uvicorn (all Python services)
- **Database**: Shared PostgreSQL server, per-service databases. SQLite acceptable for local dev/testing.
- **Migrations**: Alembic
- **Auth**: JWT access tokens + hashed refresh tokens
- **App-to-App Auth**: `X-Jarvis-App-Id` + `X-Jarvis-App-Key` headers
- **Node Auth**: `X-API-Key` header (node_id:node_key format)
- **Containerization**: Docker + docker-compose
- **Service Discovery**: jarvis-config-service with jarvis-config-client
- **Settings**: Non-sensitive config belongs in database via jarvis-settings-client. Env vars should ONLY hold sensitive values (auth keys, app IDs) and the `JARVIS_CONFIG_URL` for bootstrap discovery.

## Service Dependencies (Common Base)

Almost all backend services depend on these four:
- **jarvis-auth** - Authentication (app-to-app credentials)
- **jarvis-config-service** - Service discovery (find other service URLs)
- **jarvis-logs** - Centralized logging
- **jarvis-settings-client** - Runtime configuration

Beyond that, dependencies are service-specific (see each service's .mdc file).

## Coding Style

### Python

- **Imports**: ALL imports MUST be at the top of the file. Group: stdlib → third-party → local
- **Type hints**: ALWAYS use type hints for parameters, return types, and variable declarations
- **Prefer** `X | None` over `Optional[X]` (Python 3.10+)
- **Logging**: ALL logging MUST go through `jarvis-log-client` / `JarvisLogger`. No `print()` for logging
- **Exceptions**: Use specific exception types. Never bare `except:`. Always `except SomeException as e:`

### TypeScript/React (jarvis-admin, jarvis-installer)

- React 19, TypeScript, Tailwind CSS v4
- TanStack Query for data fetching

## Development Rules

### TDD is Mandatory

1. **RED** - Write a failing test that defines expected behavior
2. **GREEN** - Write minimum code to make it pass
3. **REFACTOR** - Clean up while keeping tests green

### Running Services

**Standard for backend services:**
```bash
./run.sh --docker              # Start in Docker
./run.sh --docker --rebuild    # Rebuild after dependency/Dockerfile changes
```

**Exceptions (macOS only - need native access to Metal GPU / Apple Vision):**
- `jarvis-llm-proxy-api` → `./run.sh` (native, Metal acceleration)
- `jarvis-ocr-service` → `./run.sh` (native, Apple Vision backend)
- On Linux, these two also use `./run.sh --docker` like everything else

**Mobile apps (React Native / Expo):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexberardi/jarvis](https://github.com/alexberardi/jarvis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
