---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

<!-- Generated: 2026-04-13 | Updated: 2026-04-13 -->

## What This Is

go-carbon is a high-performance Golang implementation of Graphite's carbon-cache daemon. It receives metrics via multiple protocols (TCP, UDP, Pickle, HTTP, Kafka, PubSub), buffers them in a sharded in-memory cache, and persists them to Whisper (.wsp) files. Also serves a read API (carbonserver) compatible with graphite-web and carbonapi.

## Build & Test Commands

```bash
make                    # Build the go-carbon binary
make test               # Run tests, vet, and race detection (all three)
go test ./...           # Run tests only (faster iteration)
go test -race ./...     # Run tests with race detector
go vet ./...            # Run vet only
go test -run TestName ./package/  # Run a single test
make image              # Build Docker image (requires Linux binary)
```

Dependencies use vendoring (`-mod=vendor` is set via GOFLAGS in the Makefile). Run `go mod tidy && go mod vendor` when updating dependencies.

## Linting

CI uses golangci-lint v2. Config is in `.golangci.yml`. Enabled linters: gocritic, ineffassign, asciicheck, misspell, promlinter, errorlint, govet, unparam, bodyclose, gochecknoinits. Formatters: gofmt, goimports.

## Key Files

| File | Description |
|------|-------------|
| `go-carbon.go` | Entry point: CLI flags, signal handling, daemon mode, starts the `carbon.App` |
| `go-carbon.conf.example` | Full annotated configuration reference (TOML format) |
| `Makefile` | Build, test, lint, packaging, Docker image targets |
| `go.mod` | Go module definition (`github.com/go-graphite/go-carbon`) |
| `.golangci.yml` | golangci-lint v2 config with enabled linters |
| `nfpm.yaml` | Package builder config for deb/rpm generation |
| `Dockerfile` | Production container image |
| `Dockerfile.debug` | Debug container with Delve support |

## Architecture

### Data Flow
```
Receivers (TCP/UDP/Pickle/HTTP/Kafka/PubSub)
    │
    ▼
  Cache (sharded concurrent map, 1024 shards)
    │
    ├──▶ Persister (worker pool writes to Whisper .wsp files)
    │
    ├──▶ CarbonLink (legacy pickle protocol for cache reads)
    │
    └──▶ Carbonserver (HTTP/gRPC read API with trigram/trie index)
```

### Core Pipeline

- **`points/`** - The fundamental data type: `Points{Metric string, Data []Point}` where `Point{Value float64, Timestamp int64}`. Flows through the entire pipeline.

- **`receiver/`** - Plugin-based metric ingestion. Each protocol (TCP, UDP, Pickle, HTTP, Kafka, PubSub) registers via `receiver.Register()` and is instantiated at startup. Protocols are wired in `carbon/app.go` with `registerPluginsOnce`. Custom receivers are configured via TOML `[[receiver.*]]` sections.

- **`cache/`** - Sharded concurrent map (1024 shards) that buffers incoming points. Three write strategies: `max` (most unwritten points first), `sorted` (by timestamp), `noop` (unspecified order). The `WriteoutQueue` feeds metrics to the persister. Optional bloom filter for new metric detection.

- **`persister/`** - Writes cached points to Whisper `.wsp` files. Configurable worker count (metrics sharded by `crc32(metricName) % workers`). Handles storage-schemas.conf and storage-aggregation.conf parsing. Supports online migration of Whisper file configurations.

- **`carbonserver/`** - HTTP+gRPC read API serving find/render/info requests to graphite-web or carbonapi. Two index types: **trigram** (default, uses trigram indexing for glob matching) and **trie** (better for 10M+ metrics, uses DFA-based matching). Supports query/find/glob caches, rate limiting, and quota enforcement.

### Supporting Packages

- **`carbon/`** - Application orchestrator. `App` struct owns all components and manages lifecycle (start, stop, config reload). `Config` is TOML-based (parsed via `BurntSushi/toml`).
- **`api/`** - gRPC API for cache queries (CarbonLink-like).
- **`tags/`** - Tag normalization and external TagDB integration.
- **`helper/`** - Protobuf definitions (`carbonpb/`, `carbonzipperpb/`), stats, throttling, file utilities.

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `carbon/` | Application orchestrator — `App` struct, config parsing, component lifecycle (see `carbon/AGENTS.md`) |
| `receiver/` | Plugin-based metric ingestion for all protocols (see `receiver/AGENTS.md`) |
| `cache/` | Sharded in-memory metric buffer between receivers and persister (see `cache/AGENTS.md`) |
| `persister/` | Whisper file writer with schema/aggregation matching (see `persister/AGENTS.md`) |
| `carbonserver/` | HTTP+gRPC read API for find/render/info with trigram and trie indexes (see `carbonserver/AGENTS.md`) |
| `points/` | Core data types: `Point` and `Points` used throughout the pipeline (see `points/AGENTS.md`) |
| `api/` | gRPC API for cache queries (CarbonLink-like protocol) (see `api/AGENTS.md`) |
| `helper/` | Shared utilities: protobuf defs, stats, throttling, file ops (see `helper/AGENTS.md`) |
| `tags/` | Graphite tag normalization and external TagDB integration (see `tags/AGENTS.md`) |
| `deploy/` | Systemd service, init script, logrotate, default config files for packaging |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go-graphite/go-carbon](https://github.com/go-graphite/go-carbon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
