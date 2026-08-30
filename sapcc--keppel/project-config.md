---
trigger: always_on
description: Guidance for AI coding agents (and humans) working in this repository. This is
---

# AGENTS.md

Guidance for AI coding agents (and humans) working in this repository. This is
the shared, tool-agnostic source of truth: Claude Code loads it through
`CLAUDE.md` (which imports this file), and other agents (Codex, Cursor, Aider,
Zed, …) read `AGENTS.md` directly. Edit repository guidance here, not in
`CLAUDE.md`.

## Repository

go-redis is the official Redis client for Go. Module path:
`github.com/redis/go-redis/v9` (Go 1.24+). The repo is a multi-module workspace
— every directory containing a `go.mod` is built and tested independently:

- root (`github.com/redis/go-redis/v9`) — the client library.
- `extra/redisotel`, `extra/redisotel-native`, `extra/redisprometheus`,
  `extra/rediscensus`, `extra/rediscmd` — instrumentation adapters with their
  own module paths (so they can pin large telemetry deps without forcing them on
  root consumers).
- `internal/customvet` — custom `go vet` analyzers (also its own module).
- `maintnotifications/e2e`, `doctests`, `fuzz`, examples under `example/` —
  separate modules.

The Makefile iterates over every `go.mod` (`GO_MOD_DIRS`) when running
`test.ci`, `go_mod_tidy`, etc. When you add a dependency in one module, you
almost never need to update the others.

## Common commands

Tests run against a Redis stack started via Docker Compose. Profiles in
`docker-compose.yml` control which services come up (`standalone`, `cluster`,
`sentinel`, `all`, `e2e`).

```sh
make docker.start                # bring up the full test stack (profile: all)
make docker.stop
make test                        # docker.start -> test.ci -> docker.stop
make test.ci                     # run tests assuming containers are already up
make test.ci.skip-vectorsets     # when REDIS_VERSION < 8
make bench                       # go test -bench=. (root module only)
make fmt                         # gofumpt + goimports -local github.com/redis/go-redis
make build
make go_mod_tidy                 # go mod tidy across every module
```

E2E (maintenance notifications) needs the extra `cae-resp-proxy` service:

```sh
make test.e2e                    # starts e2e profile, runs ./maintnotifications/e2e/, tears down
make test.e2e.docker             # subset that runs inside docker
make test.e2e.logic              # logic-only tests, no proxy required
```

Run a single test. The root suite is Ginkgo-based (`bsm/ginkgo` + `bsm/gomega`
forks), so `go test -run` matches the Go-level wrapper and you focus a spec with
the Ginkgo flag:

```sh
go test -run TestGinkgoSuite . -ginkgo.focus="ZAdd"
go test -run TestGinkgoSuite . -ginkgo.focus="cluster"
```

Plain `go test` tests (most files outside the Ginkgo suite, e.g. `internal/...`,
`maintnotifications/...`) work the usual way:

```sh
go test -run TestConnStateMachine ./internal/pool/...
go test -race -run TestCircuitBreaker ./maintnotifications/...
```

Env knobs (passed through the Makefile):

- `REDIS_VERSION` — e.g. `8.8`. Drives both the test image tag and
  `main_test.go` version-gating (`SkipBeforeRedisVersion` /
  `SkipAfterRedisVersion`).
- `CLIENT_LIBS_TEST_IMAGE` — full image ref, e.g.
  `redislabs/client-libs-test:8.8-m03`.
- `RE_CLUSTER=true` — run against a Redis Enterprise cluster instead of the
  docker-compose stack (the suite then skips ring/sentinel/TLS-cluster setup).
- `RCE_DOCKER=true` — Redis CE in docker (default for `make test`).
- `REDIS_PORT` — override the default standalone port (`6380`).

CI also runs the custom vet tool:
`go vet -vettool ./internal/customvet/customvet ./...`. The `setval` analyzer
requires every `Cmder` with a `Result()` to also have a `SetVal()`.

## Architecture

### Client types (root package)

All clients are in the root package and share most plumbing:

- `Client` (`redis.go`) — single-node client.
- `ClusterClient` (`osscluster.go`) — Redis Cluster aware. `osscluster_router.go`
  routes commands to the right shard; `internal/routing/` handles cluster-wide
  aggregation policies (e.g. fan-out for `KEYS`, `DBSIZE`).
- `Ring` (`ring.go`) — client-side sharding across independent Redis nodes
  (consistent hashing, no cluster protocol).
- Failover client (`sentinel.go`) — Sentinel-managed failover.
- `UniversalClient` (`universal.go`) — wrapper that picks one of the above based
  on options.

Command surface lives in topical files: `string_commands.go`, `hash_commands.go`,
`stream_commands.go`, `search_commands.go`, `vectorset_commands.go`, etc. Each
file defines methods on the shared `Cmdable` interface so every client type gets
the same API.

### Hooks (`redis.go` `hooksMixin`)

Three hook chains run around every operation: `DialHook`, `ProcessHook`,
`ProcessPipelineHook`. Hooks are registered via `client.AddHook(...)` and chain
in FIFO order; each hook must call `next` to continue. When a hook wraps an
error, it must call `cmd.SetErr(wrappedErr)` so the typed-error helpers
(`redis.IsLoadingError`, `IsMovedError`, etc. in `error.go`) keep working through
`errors.As`. The README has a longer pipeline-hook example.

### Connection pool (`internal/pool`)

Owns dialing, idle/active connection bookkeeping, conn state (`conn_state.go`),
pubsub-conn lifecycle (`pubsub.go`), and the dial-retry/backoff logic that powers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sapcc/keppel](https://github.com/sapcc/keppel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
