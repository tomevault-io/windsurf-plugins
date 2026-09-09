---
trigger: always_on
description: This file gives AI coding agents the repository-specific context needed to make safe, reviewable changes. It applies to the entire repository unless a deeper directory contains its own `AGENTS.md`.
---

# AGENTS.md

This file gives AI coding agents the repository-specific context needed to make safe, reviewable changes. It applies to the entire repository unless a deeper directory contains its own `AGENTS.md`.

## 1. Project summary

Page Spy API is the Go backend for [Page Spy Web](https://github.com/HuolalaTech/page-spy-web). It provides:

- HTTP APIs for rooms and debug logs.
- WebSocket-based real-time room messaging.
- SQLite or MySQL metadata persistence.
- Local filesystem or S3-compatible log-body storage.
- Internal JSON-RPC and HTTP proxying for multi-instance deployments.
- Optional hosting of the Page Spy Web frontend.

The module path is:

```text
github.com/HuolalaTech/page-spy-api
```

The repository uses Go `1.23` and declares the `go1.23.5` toolchain.

## 2. Read these documents first

English is the default documentation language:

- `README.md`: project overview and quick start.
- `docs/USAGE.md`: configuration, HTTP API, WebSocket protocol, and deployment.
- `docs/DEVELOPMENT.md`: build, test, integration-test, and troubleshooting workflows.
- `docs/ARCHITECTURE.md`: dependency graph, data flow, room model, RPC, and storage design.

Chinese equivalents use the `_ZH.md` suffix.

When implementation and documentation disagree, verify behavior from code and update both language versions in the same change.

## 3. Repository map

| Path | Responsibility |
| --- | --- |
| `api/event` | Shared event address, package, listener, and emitter contracts. |
| `api/room` | Shared room, connection, message, error, and interface contracts. |
| `bin` | Integrated executable that embeds `bin/dist/*`. |
| `config` | Configuration models, defaults, file loading, and auth environment overrides. |
| `container` | `go.uber.org/dig` dependency graph. |
| `data` | GORM models and SQLite/MySQL implementation. |
| `event` | Local event bus and remote event RPC adapter. |
| `proxy` | Cross-node HTTP reverse proxy. |
| `room` | Local rooms, remote rooms, managers, and room RPC services. |
| `rpc` | Node identity, RPC server/client, and cluster result aggregation. |
| `serve/middleware` | Request logging, CORS, authentication, errors, and cache. |
| `serve/route` | HTTP routes and log-domain orchestration. |
| `serve/socket` | WebSocket upgrade, sessions, and message loops. |
| `storage` | Local file and S3-compatible storage implementations. |
| `task` | Periodic background tasks. |
| `test/docker` | MySQL/Adminer manual integration environment. |
| `test/websocket_event_test` | Browser WebSocket smoke-test page. |

## 4. Before making changes

1. Run `git status --short`.
2. Preserve unrelated user changes; never reset or overwrite them.
3. Read the implementation, its interfaces, and all callers before changing a contract.
4. Identify whether the behavior is local-only or participates in multi-node RPC/proxy flows.
5. Check whether the change affects HTTP, WebSocket, JSON, database, storage, or configuration compatibility.
6. Plan tests before changing concurrent room, event, task, or socket code.

Prefer the smallest change that completely fixes the problem. Do not combine unrelated cleanup with a functional fix.

## 5. Build and test commands

### 5.1 Frontend embed requirement

`bin/main.go` contains:

```go
//go:embed dist/*
```

Commands that load `./bin` fail when `bin/dist` has no regular files. `bin/dist` is generated/ignored content and must not be committed.

With real frontend assets:

```bash
go build -o page-spy-api ./bin
go test ./...
go test -race ./...
go vet ./...
```

Without frontend assets, validate backend packages explicitly:

```bash
go test \
  ./api/... \
  ./config \
  ./container \
  ./data \
  ./event \
  ./logger \
  ./metric \
  ./proxy \
  ./room \
  ./rpc \
  ./serve/... \
  ./state \
  ./static \
  ./storage \
  ./task \
  ./util
```

For concurrency-related changes, run the same list with `go test -race`. Run the same list with `go vet` for static analysis.

Format changed Go files:

```bash
gofmt -w path/to/file.go path/to/file_test.go
```

Always run:

```bash
git diff --check
```

Do not claim `go test ./...` passed if it failed only because `bin/dist/*` is missing. Report that limitation and the backend-package result separately.

## 6. Runtime and configuration rules

- The process reads `config.json` from its current working directory.
- The current implementation does not support a `-c` configuration flag.
- Missing `config.json` is created from `config/defaultConfig.json`.
- Authentication environment variables are `AUTH_PASSWORD`, `JWT_SECRET`, and `JWT_EXPIRATION_HOURS`.
- Authentication environment values may be written back to `config.json`.
- Never commit real passwords, JWT secrets, MySQL DSNs, S3 credentials, or production addresses.
- Local runtime data is stored in `data/data.db`, `data.db`, and `log/`.
- The presence of `storageConfig`, even if empty, selects remote S3 storage.
- A non-empty `databaseConfig.mysqlUrl` selects MySQL; otherwise SQLite is used.

Configuration changes are compatibility-sensitive. Preserve existing JSON field names and defaults unless the task explicitly includes a migration.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HuolalaTech/page-spy-api](https://github.com/HuolalaTech/page-spy-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
