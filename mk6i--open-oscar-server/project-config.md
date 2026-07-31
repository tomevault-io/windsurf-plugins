---
trigger: always_on
description: Open OSCAR Server is an open-source instant messaging server written in Go that
---

# AGENTS.md

## Project overview

Open OSCAR Server is an open-source instant messaging server written in Go that
is compatible with classic AIM and ICQ clients. It implements the OSCAR, TOC,
and Kerberos protocols, plus HTTP management and web APIs. It is independent of
AOL/Yahoo and non-commercial.

## Quick commands

| Task                | Command                                                    |
|---------------------|------------------------------------------------------------|
| Build               | `go build -o open_oscar_server ./cmd/server`               |
| Test                | `go test -race ./...`                                      |
| Lint (matches CI)   | `gofmt -s -l . && go vet ./...`                            |
| Run (dev, plain)    | `make run`                                                 |
| Run (dev, SSL)      | `make run-ssl` (+ `make run-stunnel` in a second terminal) |
| Generate config     | `make config`                                              |
| Regenerate mocks    | `mockery`                                                  |
| Build Docker images | `make docker-images`                                       |

## Architecture

The binary in `cmd/server` starts five servers concurrently via `errgroup`:

| Server   | Protocol                   | Default port                        |
|----------|----------------------------|-------------------------------------|
| OSCAR    | FLAP/BOS (binary)          | 5190 (5193 via stunnel for SSL)     |
| TOC      | TOC (text-based)           | 9898                                |
| Kerberos | Kerberos auth              | 1088                                |
| MgmtAPI  | HTTP (management)          | 8080                                |
| WebAPI   | HTTP (web AIM-style, AMF3) | 9000 (opt-in via `ENABLE_WEBAPI=1`) |

All five servers share a common dependency container (`Container` in
`cmd/server/factory.go`) that wires together config, persistence, and business
logic.

## Key packages

| Package           | Role                                                                                                                                                                                                           |
|-------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `cmd/server`      | Entry point; wires dependencies and starts all servers.                                                                                                                                                        |
| `config`          | Configuration via env vars (`envconfig`). Config files are generated from the `Config` struct—do **not** edit them by hand; run `make config` instead.                                                         |
| `foodgroup`       | Core business logic for OSCAR "food groups": Auth, Buddy, Feedbag, ICBM, Chat, BART, Locate, OService, ICQ, Admin, PermitDeny, ODir, Stats, UserLookup, ChatNav. Shared by the OSCAR, TOC, and WebAPI servers. |
| `wire`            | OSCAR wire protocol: SNAC/FLAP encoding, TLV, food group codes, rate limits, frames.                                                                                                                           |
| `state`           | Persistence and in-memory state: `SQLiteUserStore`, `InMemorySessionManager`, `InMemoryChatSessionManager`, DB migrations.                                                                                     |
| `server/oscar`    | OSCAR protocol server; SNAC routing and handler wiring.                                                                                                                                                        |
| `server/toc`      | TOC protocol server (text-based).                                                                                                                                                                              |
| `server/kerberos` | Kerberos auth server.                                                                                                                                                                                          |
| `server/http`     | Management HTTP API (users, sessions, chat rooms). Spec: `api.yml`.                                                                                                                                            |
| `server/webapi`   | Web AIM-style API (AMF3). Spec: `docs/open_api/webapi.yml`.                                                                                                                                                    |

## Database

SQLite via `modernc.org/sqlite` (pure-Go, no CGO). Default file: `oscar.sqlite`.
Migrations live in `state/migrations/` as numbered `.up.sql` / `.down.sql` pairs
and are applied automatically at startup.

## Configuration

Configuration is env-var driven via `kelseyhightower/envconfig`. Settings files
(`config/settings.env`, `config/ssl/settings.env`) are **generated** from the
`Config` struct by running `make config`. Never edit them by hand—change the
`Config` struct and regenerate.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mk6i/open-oscar-server](https://github.com/mk6i/open-oscar-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
