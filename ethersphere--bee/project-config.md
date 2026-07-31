---
trigger: always_on
description: Project instructions for **AI coding assistants and agents** (OpenAI Codex, Cursor, GitHub Copilot, Claude Code, and similar tools). This file is the canonical source of shared project instructions; `CLAUDE.md` imports this file for Claude Code.
---

# AGENTS.md

Project instructions for **AI coding assistants and agents** (OpenAI Codex, Cursor, GitHub Copilot, Claude Code, and similar tools). This file is the canonical source of shared project instructions; `CLAUDE.md` imports this file for Claude Code.

## Project overview

Bee is the reference Go implementation of an Ethereum Swarm node. It implements decentralized storage and communication: content-addressed chunk storage, Kademlia-based routing, postage stamp accounting, push/pull syncing, PSS messaging, feeds, and storage incentives (redistribution game).

**Module**: `github.com/ethersphere/bee/v2`

**Go version**: 1.26 (see `go.mod`)

**License**: BSD 3-clause (see `LICENSE`)

Human-oriented contributing docs: `CONTRIBUTING.md`, `CODING.md`, `CODINGSTYLE.md`, `README.md`.

## Guidelines

Keep changes **minimal and focused**. Only touch code that belongs to the task. Do not refactor unrelated code, rename symbols for style only, or mix unrelated fixes in one commit or PR.

Read **`CONTRIBUTING.md`**, **`CODING.md`**, and **`CODINGSTYLE.md`** for process, patterns, and style. Prefer matching existing naming, types, imports, and log style in the files you edit.

Do **not** add, remove, or update `go.mod` dependencies unless the task **explicitly** requires it or the person asking for the work **explicitly** requests a dependency change.

Handle errors and logging the way this repo does: propagate errors with context (`fmt.Errorf("…: %w", err)`), avoid logging and returning the same error, and use structured logging with clear operator vs developer levels (see `CODING.md`).

Prefer **`package foo_test`** tests, **`export_test.go`** when you must export internals, and **`t.Parallel()`** only where it is safe. Add or update tests when behavior changes. Integration tests use **`-tags=integration`**.

## Pre-commit checklist

Before you finish a change set (especially before a commit or PR), run these and fix failures:

1. **Formatting** — `make format` (gofumpt + gci; see `CODING.md`).
2. **Compile** — `make build` (all packages) and, when you need the binary artifact, `make binary` (`dist/bee`, `CGO_ENABLED=0`).
3. **Tests** — `make test` (unit tests, `-failfast`). For a single package use `go test ./pkg/<name>/...`. Use `make test-race` when concurrency is central to the change. Use `make test-integration` only when you touch integration-tagged code.
4. **Static checks** — `make lint` and `make vet` (see `.golangci.yml`).

CI pipelines may use `make test-ci` / `make test-ci-race` (see `Makefile` for flags).

## Dev commands (quick reference)

```bash
make binary     # dist/bee
make build      # compile all packages
make test       # unit tests
make test-race  # unit tests + race detector
make lint       # golangci-lint (see .golangci.yml)
make vet        # go vet
make protobuf   # regenerate *.pb.go after changing .proto files
```

## Architecture

### Entry point and CLI

Binary built from `cmd/bee/main.go`. CLI uses Cobra + Viper:

- `bee start` — full or light node (`cmd/bee/cmd/start.go`)
- `bee init` — initialize data directory
- `bee deploy` — deploy smart contracts
- `bee db` — database management
- `bee version` — print version info

Configuration: option constants in `cmd/bee/cmd/cmd.go`. Viper reads CLI flags, environment variables (`BEE_` prefix), and YAML config.

### Node bootstrap

`pkg/node/node.go` is the main orchestrator. `NewBee()` wires subsystems via dependency injection; avoid global mutable state. The `Bee` struct holds service references and provides `Shutdown()` for teardown.

### HTTP API

- Router: `gorilla/mux` in `pkg/api/router.go`
- Route groups in `Mount()`:
  - `mountTechnicalDebug()` — `/node`, `/addresses`, `/health`, `/readiness`, `/metrics`, `/loggers`, pprof
  - `mountBusinessDebug()` — topology, accounting, settlements, stamps management
  - `mountAPI()` — `/bytes`, `/chunks`, `/bzz`, `/feeds`, `/soc`, `/stamps`, `/tags`, `/pins`, `/pss`, `/grantee`
- `checkRouteAvailability` can block endpoints during sync
- OpenAPI: `openapi/Swarm.yaml` (API versioning follows SemVer there; the main Bee release version does not)
- Endpoints exist at root (e.g. `/bytes`) and under `/v1/` (e.g. `/v1/bytes`)

### P2P networking

- Transport: libp2p (`pkg/p2p/libp2p/`)
- Wire formats: protobuf (gogo) — each protocol area has a `pb/` directory with `.proto` and `doc.go` (`go:generate` calling `protoc` + `--gogofaster_out`)
- Important protocol packages: `pushsync`, `pullsync`, `retrieval`, `pingpong`, `hive`, `pricing`

### Storage

- Chunk types: CAC (`pkg/cac/`), SOC (`pkg/soc/`)
- Interfaces: `pkg/storage/` (`Putter`, `Getter`, `Hasser`, `Deleter`)
- Local store: `pkg/storer/` (reserve, cache, upload, pinning)
- Blob engine: `pkg/sharky/`
- BMT: `pkg/bmt/`
- State: `pkg/statestore/` (LevelDB); `pkg/shed/` (typed LevelDB layer)

### Postage and incentives

- `pkg/postage/` — batches, stamps, services
- `pkg/postage/listener/` — on-chain events
- `pkg/postage/postagecontract/` — contract interaction
- Stamps: batch ID, depth (capacity), amount (per-chunk value)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ethersphere/bee](https://github.com/ethersphere/bee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
