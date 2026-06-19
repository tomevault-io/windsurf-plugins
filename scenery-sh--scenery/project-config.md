---
trigger: always_on
description: Use when building, running, debugging, inspecting, validating, or generating clients for scenery applications. scenery is a Go-native service runtime and CLI using .scenery.json, //scenery directives, typed endpoints, local dev supervision, logs, traces, metrics, workers, and TypeScript client generation.
---


# scenery

scenery is a Go-native service runtime and local development platform. It runs app-root dev runtimes, exposes capabilities for inspection and action, and hides backing substrate details unless you intentionally debug them. Apps are ordinary Go modules with a `.scenery.json` app root and `//scenery:` directives in Go source.

This skill is the portable agent entrypoint. It teaches shared scenery behavior, but it does not replace app-local instructions. Client apps should also keep a small `AGENTS.md` with app root, frontend roots, generated client paths, required environment names, validation commands, and product invariants. In target apps, read the root `AGENTS.md` and every child `AGENTS.md` on the path to files you expect to touch before editing non-trivial changes.

Read next when needed:

- `docs/agent-guide.md` for agent workflow, capabilities, generated artifacts, and client-app integration.
- `docs/local-contract.md` for exact CLI grammar, JSON schemas, artifact paths, and stability labels.
- `docs/app-development-cookbook.md` for app recipes.
- `docs/ui-agent-contract.md` before UI work.

## Agent Fast Path

```sh
scenery check --json
scenery doctor --json
scenery inspect app --json
scenery inspect routes --json
scenery inspect endpoints --json
scenery inspect models --json
scenery inspect views --json
scenery inspect wire --json
scenery system toolchain verify --json
scenery logs --jsonl --limit 200
scenery inspect observability --json
scenery logs query --json --since 15m --query 'error OR panic'
scenery harness --json --write
scenery validate quick --json --write
```

Prefer JSON output for agent decisions. Prefer `scenery up` for local development. Use `scenery serve` for headless API execution. Use `scenery task` for configured and code tasks. Use `scenery validate` for app-owned quality gates. Use `scenery worker` for worker-only cron/Temporal execution.

Run `scenery doctor --json` before deep app debugging when local readiness is in doubt. It is read-only and reports host resources, Go version, Docker engine reachability/details, optional tools, and app-sensitive dependency hints without building or starting services.

`scenery inspect docs --json` exposes `summary.review_due_count`, document-level `review_due` and `stale` fields, discovered `AGENTS.md` scopes, and Child Agent Index drift. For scenery repo changes, `scenery harness self --summary --write` surfaces those docs knowledge signals in compact validation summaries and leaves full evidence in `.scenery/harness/` artifacts. When docs and behavior disagree, the same PR must either fix the affected docs or open/update an ExecPlan that records the drift.

## Mental Model

- `.scenery.json` marks the app root.
- App-required Go build tags or build-time flags belong in `.scenery.json` as `build.go_flags`, for example `["-tags=roofmapnet_native"]`; Scenery applies them to app builds and generated-workspace tests.
- Go source is the app model.
- `scenery up` starts the supervised local platform: app process, rebuild/restart loop, dashboard, API Explorer, logs, traces, metrics, managed dev services when configured, and optional frontend routing through the local agent.
- `scenery serve` starts a headless API-role server and does not start dashboard, proxy, or watch mode.
- Public and auth endpoints are externally reachable. Private endpoints are internal-only and called through generated helpers.
- Typed endpoints decode path, query, header, cookie, and JSON body inputs into Go values.
- Generated internal calls preserve routing, private access, auth context, tracing, and error semantics.

## Minimal App

```json
{"name":"hello"}
```

```go
module example.com/hello

go 1.26.3

require scenery.sh v0.0.0
```

```go
package service

import "context"

type HelloResponse struct {
	Message string `json:"message"`
}

//scenery:api public path=/hello/:name method=GET
func Hello(ctx context.Context, name string) (*HelloResponse, error) {
	return &HelloResponse{Message: "hello " + name}, nil
}
```

```sh
scenery check --json
scenery serve
curl http://127.0.0.1:4000/hello/world
```

## Directives

```go
//scenery:api public|auth|private [raw] [path=/...] [method=GET,POST]
//scenery:service
//scenery:authhandler
```

Standard auth can be enabled from `.scenery.json` without app-local wrapper endpoints. Its tenant tables are framework-owned in PostgreSQL schema `scenery_auth` including `scenery_auth.tenants`; app-local `tenants` services or tables are product-domain concerns only.

Typed endpoint shape:

```go
func Endpoint(ctx context.Context, pathParam string, req *Request) (*Response, error)
```

Raw endpoint shape:

```go
func Endpoint(w http.ResponseWriter, req *http.Request)
```

Struct tags:

- request decoding: `json`, `header`, `query`, `qs`, `cookie`
- scenery tags: `scenery:"optional"`, `scenery:"httpstatus"`

## Public Go Packages

- `scenery.sh`
- `scenery.sh/auth`
- `scenery.sh/errs`
- `scenery.sh/middleware`
- `scenery.sh/temporal`
- `scenery.sh/cron`
- `scenery.sh/pgxpool`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scenery-sh/scenery](https://github.com/scenery-sh/scenery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
