---
trigger: always_on
description: The repository is a single Go module (`github.com/sacloud/sakumock`); each service is a package under its own subdirectory. Shared building blocks live in the `core/` package at `github.com/sacloud/sakumock/core` — the `Route` type and `PrintRoutes` formatter, the CLI serve helpers (`Serve`, `NotifyContext`, `SetupLogger`, `RateLimitHint`), the common TLS support (`TLSFiles` + `ServeListener` + `WithTLSScheme`, see "TLS" below), and the `IDGenerator` for sequential numeric resource IDs (services
---

# sakumock

## Service Conventions

The repository is a single Go module (`github.com/sacloud/sakumock`); each service is a package under its own subdirectory. Shared building blocks live in the `core/` package at `github.com/sacloud/sakumock/core` — the `Route` type and `PrintRoutes` formatter, the CLI serve helpers (`Serve`, `NotifyContext`, `SetupLogger`, `RateLimitHint`), the common TLS support (`TLSFiles` + `ServeListener` + `WithTLSScheme`, see "TLS" below), and the `IDGenerator` for sequential numeric resource IDs (services pass their own base value). Services must not import each other; shared code goes through `core`.

All services are also aggregated into a single `sakumock` binary (entrypoint `cmd/sakumock`). It exposes each service as a subcommand (`sakumock <service>`) with the same flags as the standalone `sakumock-<service>` binary. Only this unified binary is released as a prebuilt artifact (via GoReleaser); per-service binaries remain `go install`-able. See "Unified binary & release" below.

### Required public API

- `Config` struct with `alecthomas/kong` tags
- `Config.ClientEnv() []core.EnvVar` — the `SAKURA_ENDPOINTS_*` override(s) a client (SDK / Terraform provider) sets to reach this mock, derived from `Config.Addr`. This is the single source of truth for the service's endpoint env var name(s): both the standalone startup log and the unified binary's `env` subcommand derive from it. SimpleMQ returns two (control-plane `_QUEUE` and data-plane `_MESSAGE`, both on its single address); most services return one.
- `Config` must also satisfy `core.ServiceConfig` (`Name() string`, `ListenAddr() string`, `ClientEnv()`, `NewServer(core.ServerOptions) (core.Server, error)`) — the interface the unified binary loops over to build and describe every service without hard-coding per-service names/addresses/endpoints. `Name()` returns the service's short name (matching the subcommand, e.g. `"simplemq"`); `ListenAddr()` returns `Config.Addr`; `NewServer(opts)` applies the shared `opts` (currently `opts.IDGen`) and adapts `NewHandler`. `core.ServerOptions` carries shared dependencies the unified binary injects into every service — add a field there to reach all services without changing the interface. Asserted with `var _ core.ServiceConfig = Config{}` in `server.go`.
- `Command` struct — embeds `Config`, adds a `Routes bool` flag and a `Run(ctx context.Context) error` method; reused by both the standalone `sakumock-<service>` binary and the unified `sakumock` binary
- `NewHandler(cfg Config) (*Server, error)` — creates `http.Handler` without listener (return a `nil` error when construction cannot fail, to keep the signature uniform across services)
- `NewTestServer(cfg Config) *Server` — creates and starts `httptest.Server` (panics on `NewHandler` error)
- `Server.TestURL() string` — returns base URL
- `Server.Routes() []core.Route` — returns metadata for every HTTP endpoint registered on the server (the CLI's `--routes` flag prints these via `core.PrintRoutes`)
- `Server.Close()` — shuts down server and releases resources
- `*Server` must satisfy `core.Server` (`http.Handler` + `Routes()` + `TestURL()` + `Close()`), the interface the unified binary uses to treat every service uniformly. Each service asserts it with a compile-time `var _ core.Server = (*Server)(nil)` in `server.go`, so a new service that drifts from the contract fails to build (the convention is type-enforced, not just documented).

### File structure

- `store.go` — Store interface and domain types
- `store_memory.go` — in-memory Store implementation
- `new_store.go` — Store factory
- `handler.go` — HTTP handlers and JSON types
- `route.go` — `routeTable()` (single source of truth driving both `buildMux()` and `Routes()`) plus the public `Routes()` method, all built on the shared types in `github.com/sacloud/sakumock/core`
- `validate_gen.go` + `generate.go` — generated spec-derived request-body validation table and its `//go:generate` directive (see "Spec-derived request-body validation" below)
- `server.go` — Config, Server, NewHandler, NewTestServer
- `cli.go` — `Command` (embeds `Config`, adds `--routes`); its `Run` sets up logging, prints routes or starts the server via `core.Serve`, and holds the service-specific startup log lines
- `cmd/sakumock-<service>/` — standalone CLI entrypoint; a thin shim that parses flags into `Command` and calls `Command.Run` (uses `core.NotifyContext` for signal handling — no per-service signal files)
- Makefile, README.md

There is no per-service version: every binary reports `sakumock.Version` from the repository-root `version.go` (package `sakumock`), kept in sync with the git tag by tagpr (root `.tagpr`).

### Mock-only endpoints (`/_sakumock/`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sacloud/sakumock](https://github.com/sacloud/sakumock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
