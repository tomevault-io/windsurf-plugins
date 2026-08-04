---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OPRAI is a DeFi-native conversational AI assistant for Solana. It transforms natural language into on-chain actions (swaps, token launches, staking, transfers) with wallet-based authentication (SIWS).

## Common Commands

### Setup & Run (Polyglot — primary)
```bash
cp .env.example .env              # Pre-filled with generated secrets; add your own OpenAI key
make install                      # Install all deps (Node.js + Go + Rust + Python)
make proto                        # Generate gRPC stubs from proto/ (Go, Python, Rust)
make build-python                 # Create Python venvs and install deps
make dev-infra                    # Start Postgres (5433), Redis (6379), Qdrant (6333) in Docker
make migrate                      # Run database migrations
make dev-all                      # Start infra + all 7 polyglot services via honcho
make health                       # Check gateway aggregated health
```

### Individual Service Groups
```bash
make dev-go                       # Gateway + Auth + Admin (Go)
make dev-rust                     # Solana service (Rust)
make dev-python                   # Chat + Memory (Python, uvicorn --reload)
make dev-angular                  # Angular frontend on :3000
```

### Build
```bash
make build-go                     # Compile all Go services
make build-rust                   # cargo build --release for Rust
make build-angular                # ng build --configuration production
make build-all                    # proto + all services + frontend
```

### Legacy Node.js Stack (still compiles, gradually being replaced)
```bash
pnpm install && pnpm build        # Build all legacy packages/services via Turborepo
pnpm dev                          # Start legacy services (excludes oprai-web)
pnpm test                         # Run all Vitest tests (legacy)
pnpm --filter @oprai/auth-service dev        # Single legacy service
pnpm --filter @oprai/auth-middleware test     # Single legacy package test
pnpm --filter @oprai/chat-service db:migrate # Single service migrations
pnpm dev:admin                    # Legacy admin-service + admin-panel only
```

### Database
```bash
make migrate                      # All service migrations
make backup                       # pg_dump to backups/
make restore                      # Restore from latest (or BACKUP=path)
make reset                        # Drop + recreate (creates backup first, requires confirmation)
```

### Docker (Full Stack)
```bash
make docker-up                    # Build + start full polyglot stack with monitoring
make docker-down                  # Stop
make docker-logs                  # Tail logs
docker compose up -d              # Legacy Node.js stack only
```

## Architecture

**Dual-stack monorepo**: polyglot services (primary) coexist with legacy Node.js services during migration. Inter-service communication is **gRPC + Protobuf** (15 proto files under `proto/`). Monitoring via **Prometheus** (:9090) + **Grafana** (:3333).

```
Frontend (Angular :3000) → Bearer JWT → Gateway (Go :3001)
                                          ├── auth-service (Go :3010/50051)    → auth_schema
                                          ├── chat-service (Python :3020/50052) → chat_schema
                                          ├── solana-service (Rust :3030/50053) → solana_schema
                                          └── memory-service (Python :3040/50054) → memory_schema

Admin Panel (Angular :3000) → Bearer JWT → admin-service (Go :3050/50055) → cross-schema SQL
```

### Polyglot Services (new, primary)
| Service | Path | Language | Framework | Port (HTTP/gRPC) |
|---------|------|----------|-----------|------------------|
| Gateway | `services/gateway-go/` | Go | Chi, gobreaker | 3001 |
| Auth | `services/auth-service-go/` | Go | Chi, pgx, golang-jwt, go-redis | 3010 / 50051 |
| Chat | `services/chat-service-py/` | Python | FastAPI, LangChain, SQLAlchemy | 3020 / 50052 |
| Solana | `services/solana-service-rs/` | Rust | Actix-Web, Tonic, solana-sdk, Diesel | 3030 / 50053 |
| Memory | `services/memory-service-py/` | Python | FastAPI, qdrant-client, OpenAI | 3040 / 50054 |
| Admin | `services/admin-service-go/` | Go | Chi, sqlc, bcrypt | 3050 / 50055 |
| Frontend | `apps/oprai/` | TypeScript | Angular 19, standalone components | 3000 |
| OpraiOS | `opraios/` | Python | Pydantic, OpenAI, solana-py | standalone |

### Additional Services (newer, not yet in the table above)
- **`services/solana-service-ts/`** — TypeScript rewrite of the Rust solana-service using official protocol SDKs (Jupiter, Kamino, Drift, Marinade, Meteora, MarginFi). Runs alongside the Rust service: `make dev-solana-ts` (PORT 3030) or via Procfile as `solana-ts` (PORT 3031). The two are interchangeable backends for the same `/actions/*` surface — the Rust service is still the documented default; the TS one is the migration target.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Oprai-Labs/oprai](https://github.com/Oprai-Labs/oprai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
