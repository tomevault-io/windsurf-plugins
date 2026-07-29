---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Celestia Indexer — CLAUDE.md

## Project Overview

Go-based blockchain indexer + REST API for the Celestia Data Availability (DA) blockchain. Indexes blocks, transactions, messages, events, blobs, namespaces, validators, governance, IBC, Hyperlane, and rollups into PostgreSQL (TimescaleDB) and exposes them via public Echo HTTP/WebSocket API and a private admin API.

## Architecture

```
cmd/
  indexer/              # Core indexer daemon
  api/                  # Public REST API (port 9876)
    handler/            # Echo handlers (one file per entity)
    handler/responses/  # DTO structs for API responses
  private_api/          # Admin API (port 9877)
  celestials/           # Off-chain Celestials data indexer
pkg/
  indexer/              # Core indexer pipeline
    receiver/           # Fetches blocks from CometBFT RPC/API/WS
    parser/             # Decodes raw blocks, txs, messages, events
    storage/            # Saves parsed data to DB in one DB transaction
    rollback/           # Handles chain reorganizations
    genesis/            # Handles genesis block separately
    decode/context/     # Context object accumulates all parsed entities (passed parser → storage)
    config/             # Indexer config structures
  node/
    rpc/                # CometBFT RPC client
    api/                # Node REST API client
    dal/                # DAL API client (blob retrieval)
  types/                # pkg-level domain types (Level, etc.)
internal/
  storage/              # Domain model structs + storage interfaces (IXxx)
    id.go               # Deterministic ID generation (height<<24 | position)
    postgres/           # Bun ORM implementations of all interfaces
      scopes.go         # Reusable query filters and pagination helpers
      transaction.go    # DB transaction: save/rollback all entities
      core.go           # DB init, migrations, hypertables, enums, indexes
      migrations/       # Bun migrations (named by date)
    types/              # Enums (MsgType, EventType, ModuleType, etc.)
  blob/                 # Blob handling utilities
  pool/                 # sync.Pool wrappers for reusing slices
  currency/             # Currency utilities
  stats/                # Statistics calculations
database/
  functions/            # PostgreSQL functions (materialized view refresh)
  views/                # Materialized views for analytics (minute/hour/day/...)
configs/
  dipdup.yml            # YAML config with ${ENV_VAR:-default} substitution
```

**Indexer pipeline:** CometBFT RPC/WS → Receiver → Parser → Storage module → PostgreSQL

**Module wiring** (in `pkg/indexer/indexer.go`): modules connect via named inputs/outputs using `module.AttachTo(source, outputName, inputName)`. Every module has a `StopOutput` that feeds into the stopper.

## Key Libraries

| Purpose | Library |
|---------|---------|
| HTTP | `github.com/labstack/echo/v4` |
| ORM | `github.com/uptrace/bun` + `github.com/jackc/pgx/v5` (via `pgx/v5/stdlib`) |
| Blockchain | `github.com/celestiaorg/celestia-app/v9`, `github.com/cometbft/cometbft` |
| Cosmos | `github.com/cosmos/cosmos-sdk`, `github.com/cosmos/ibc-go/v8` |
| Cache | `github.com/valkey-io/valkey-go` |
| Logging | `github.com/rs/zerolog` |
| Validation | `github.com/go-playground/validator` |
| Errors | `github.com/pkg/errors` |
| Mocks | `go.uber.org/mock/mockgen` |
| Swagger | `github.com/swaggo/swag` |
| Indexer SDK | `github.com/dipdup-net/indexer-sdk` |
| JSON | `github.com/bytedance/sonic` |
| Profiling | `github.com/grafana/pyroscope-go` |
| Sentry | `github.com/getsentry/sentry-go` |

## Commands

```bash
make init         # chmod + run init.dev.sh (first-time dev setup)
make indexer      # go run ./cmd/indexer -c ./configs/dipdup.yml
make api          # go run ./cmd/api -c ./configs/dipdup.yml
make private_api  # go run ./cmd/private_api -c ./configs/dipdup.yml
make celestials   # go run ./cmd/celestials -c ./configs/dipdup.yml
make build        # build all binaries to /bin
make test         # go test -p 8 -timeout 120s ./...
make generate     # go generate (mocks + enums for storage, blob, node, gas packages)
make api-docs     # swag init (regenerate Swagger)
make ga           # generate + api-docs
make lint         # golangci-lint
make gc           # lint → test → commit
make compose      # docker compose up --build
make cover        # generate coverage report
make license-header  # add/update SPDX headers across all files
```

**Run a single test or package:**
```bash
go test ./internal/storage/postgres/... -run TestBlockByHeight -v
go test ./cmd/api/handler/... -timeout 30s
```

## Configuration

YAML config with `${ENV_VAR:-default}` substitution (`configs/dipdup.yml`):

```
# Datasources
CELESTIA_NODE_RPC_URL / CELESTIA_NODE_API_URL / CELESTIA_NODE_WS_URL
CELESTIA_DAL_URL / CELESTIALS_URL / CELENIUM_BLOBS_URL

# Database
POSTGRES_HOST / PORT / USER / PASSWORD / DB / MAX_OPEN_CONNECTIONS

# API
API_HOST / API_PORT / API_RATE_LIMIT / WEBSOCKET_ENABLED
PRIVATE_API_HOST / PRIVATE_API_PORT

# Cache
CACHE_URL / CACHE_TTL

# Indexer
INDEXER_START_LEVEL / INDEXER_SCRIPTS_DIR / NETWORK
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [celenium-io/celestia-indexer](https://github.com/celenium-io/celestia-indexer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
