---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository

Altair is a Go-based API gateway (entry point `altair.go`, root package `main`). It depends on the sibling `github.com/kodefluence/monorepo` library for `kontext`, `exception`, `jsonapi`, `db`, and `memorystore`. Go 1.19, Gin, zerolog, Cobra, MySQL driver.

## Commands

All commands are driven by the `Makefile`; prefer them over ad-hoc `go` invocations because `make test` excludes packages that don't contribute coverage (`altair$`, `core`, `mock`, `interfaces`, `testhelper`).

- `make test` — full test suite with race detector and `cover.out`.
- Run a single test: `go test -run TestName ./module/router/usecase/...` (pass `-race` to match CI).
- `make mock_all` — regenerate every mock (`mock_metric`, `mock_plugin`, `mock_loader`, `mock_routing` plus service/formatter/model/validator targets). Individual mocks use `mockgen -source core/<file>.go -destination mock/mock_<file>.go -package mock`; sub-package mocks are driven by `//go:generate mockgen ...` directives co-located with the interface they mock.
- Build: `make build_linux` / `build_darwin` / `build_windows` (produce UPX-packed binaries in `./build_output/<os>/altair`). `make build` runs all three.
- Docker: `make build_docker` / `build_docker_latest` / `push_docker`. The `Dockerfile` expects `build_linux` to have run first — it COPYs `./build_output/linux/altair` into the image.
- Local stack: copy `env.sample` to `.env`, then `make docker-compose-up` (MySQL 5.7 on 127.0.0.1:3306). Pair it with `altair run` in another terminal.
- Linting matches CI via `golangci-lint run` (CI pins `v1.50.1`).

## Runtime entry

`altair.go` is a Cobra root with three subcommands:
- `altair run` — loads `config/app.yml`, `config/database.yml`, `config/plugin/*`, fabricates DB connections, mounts healthcheck + plugins, serves on `appConfig.Port()` (default 1304). Graceful shutdown listens for SIGINT/SIGTERM.
- `altair config [app|db|all]` — dumps loaded config.
- `altair plugin ...` — runs plugin-provided Cobra subcommands (migrations, application creation, etc.). `DisableFlagParsing: true` so each plugin owns its flag surface.

All three share `loadConfig()` at startup, so missing `config/` files silently yield `nil` configs that each subcommand checks before executing.

## Architecture

The repo is a strict five-layer, inward-pointing design. Dependencies flow only toward `core` and `entity`; nothing in `core` or `entity` imports outward.

```
core/      — interfaces only (AppConfig, Controller, Metric, DownStreamPlugin, RouteCompiler, …)
entity/    — pure data types + constructor option structs (AppConfig, RouteObject, DBConfig)
adapter/   — wraps external concretes into core.* interfaces (entity.AppConfig -> core.AppConfig)
cfg/       — YAML loaders + "bearer" state holders; env-interpolated via text/template with {{ env "FOO" }}
module/    — feature assemblies; every feature has provider.go + usecase/ (+ controller/ when HTTP-facing)
plugin/    — versioned extensions (oauth, metric) with their own mini-layering inside
```

### Module pattern

Every module exposes a single entry: `Provide(...)` (returns a `module.*` interface) or `Load(appModule)` (mutates the controller registry). Internally:
- `module/<name>/provider.go` — thin DI: constructs usecase and returns it typed as the `module.*` interface.
- `module/<name>/usecase/` — business logic; one file per struct, each with its own `_test.go`. Usecases consume consumer-owned interfaces defined in the same file (see `module/oauth/.../application_manager.go` — `OauthApplicationRepository` and `Formatter` are declared there, not in the repo package).
- `module/<name>/controller/` — HTTP/command/downstream adapters (only for features with transport surfaces, e.g. `healthcheck`, `projectgenerator`, or inside plugins).

`module/interface.go` is the shared contract: `App`, `Controller`, `HttpController`, `CommandController`, `DownstreamController`, `MetricController`, `ApiError`, `RouterPath`.

### Request lifecycle

`module/controller/usecase/http.go` `InjectHTTP` wraps every registered `HttpController`:
1. Generates `request_id` (uuid), sets `start_time`, fabricates a `kontext.Context` wrapping the gin context.
2. Buffers request body so downstream handlers can re-read it.
3. `defer ctrl.httpRecoverFunc(...)` catches panics, emits an Internal Server Error JSON:API response, logs with structured tags.
4. Dispatches to `httpController.Control(ktx, c)`.
5. Branches on `c.Writer.Status()` to log info vs error with elapsed time.
6. Increments `controller_hits` counter + `controller_elapsed_time_seconds` histogram against every registered `MetricController`.

Controllers themselves never touch loggers or metrics — they just call their manager and `c.JSON(status, jsonapi.BuildResponse(...))`.

### Routing / proxy

`module/router/usecase/compiler.go` walks `./routes/*.{yml,yaml}` with `text/template` (`{{ env ... }}` helper) and unmarshals each into an `entity.RouteObject` containing per-path `Auth`/`Scope`. `generator.go` registers one Gin `engine.Any(urlPath, ...)` per path; at request time it:
1. Builds a `proxyReq` copy of the client request.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kodefluence/altair](https://github.com/kodefluence/altair) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
