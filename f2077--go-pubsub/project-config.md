---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`go-pubsub` is a lightweight, in-process Pub/Sub library for Go. It is pure fire-and-forget: zero persistence, no delivery guarantees, designed for transient, low-latency data flows (real-time streaming-media packets, gaming events, live signals). Full subscriber channels drop messages silently — there is no backpressure.

Module path: `github.com/F2077/go-pubsub` (Go 1.25, generics-based).
Direct dependencies: `github.com/google/uuid`, `github.com/stretchr/testify`.

## Common commands

```bash
# Build
go build ./...

# Run all unit tests in the pubsub package
go test ./pubsub/...

# Run a single test (matches -run regex against test names)
go test -run TestBasicPubSub ./pubsub/...
go test -run TestChannelOverflow ./pubsub/...

# Run benchmarks (the README's published numbers come from these)
go test -bench=. ./pubsub/...

# Run a specific benchmark, e.g.
go test -bench=BenchmarkPublishWithTimeout ./pubsub/...

# Race detector — strongly encouraged for concurrency changes
go test -race ./pubsub/...

# Vet / static check
go vet ./...

# Coverage report (one-line summary per package)
go test -cover ./pubsub/...

# Hunt for flaky tests (N iterations with race detector)
go test -count=10 -race ./pubsub/...

# Run the example program
go run ./cmd/quickstart
```

The toolchain story is `make` + CI, not just `go test`/`go vet`:
- `Makefile` — `make test` / `make bench` / `make lint` / `make cover` wrap the common commands.
- `.golangci.yml` — v2 schema; enables `govet`, `ineffassign`, `unused`, `revive`, `staticcheck`.
- `.github/workflows/test.yml` — on push/PR runs: `go mod tidy -diff`, a **gofmt gate** (`gofmt -l` must be empty — always `gofmt -w` before committing!), `go vet`, `go test -race -count=1 -coverprofile`, and `golangci-lint v2.12.2` (`only-new-issues: true`). So run `gofmt -w` + `go vet` + `go test -race` locally before pushing, or CI will go red.

The `infertypeargs` lint (e.g. `WithLogger[string]` when `T` is inferred)
fires across every file in the codebase. It is pre-existing stylistic
noise, not a correctness issue; it is explicitly excluded in
`.golangci.yml` (staticcheck "unnecessary type arguments"), so it will
not fail CI. It is out of scope for unrelated work.

## Code layout

```
pubsub/
  pubsub.go         # package marker only (single-line file)
  broker.go         # Broker[T], subscription[T], options, capacity mgmt
  publisher.go      # Publisher[T] (thin wrapper around broker.Publish)
  subscriber.go     # Subscriber[T], Subscription[T], SubscriptionOption
  coverage_test.go   # unit tests for the public surface (gap-fillers)
  concurrent_test.go # concurrency safety-net tests (run under -race)
  example_test.go    # godoc Example* tests; show up on pkg.go.dev
  perf_test.go       # zero-alloc assertions (//go:build !race)
  pubsub_test.go     # unit tests
  bench_test.go      # benchmarks cited in README
cmd/
  quickstart/main.go  # runnable "Quick Start" example
```

Everything user-facing lives in the `pubsub` package; the `cmd/quickstart` binary is the only consumer and doubles as documentation.

## Architecture

Three generic types (`T` = message payload) form the public surface:

- **`Broker[T]`** owns the topic→subscription map and the global capacity cap (default 8192 topics). It guards the map with `sync.RWMutex` and is the only place a new `subscription` is created or removed. All log output is emitted through an injected `*slog.Logger` (defaults to `slog.NewTextHandler(os.Stdout, nil)` — tests pass a stderr handler to keep bench/test output clean).
- **`Publisher[T]`** is a UUID-keyed handle on a broker. `Publish(topic, msg)` resolves the topic's subscription via `createOrLoadSubscription` and calls `subscription.deliver`.
- **`Subscriber[T]`** holds per-topic `chan T` in a `map[string]chan T` guarded by `subscriber.mutex`, plus a per-topic `*time.Timer` (created only when `WithTimeout` is set). The per-subscription `chan error` lives on each `*Subscription` (not shared, so one sub's `Close` can't close another's `ErrCh`). `Subscribe` returns a `*Subscription[T]` exposing the read-only `Ch` and `ErrCh` channels and a `Close` method.

The internal `subscription[T]` is the fan-out unit: it tracks all `Subscriber`s for one topic. On `deliver` it snapshots the subscriber set into a `sync.Pool`-recycled slice (zero allocation on the hot path — the pool stores `*[]*Subscriber[T]` so only a pointer boxes into `interface{}`), then for each subscriber it holds `subscriber.mutex` to read `channels[topic]` and non-blockingly `select`-send (`default: drop`). A successful send calls `subscriber.resetTimer` **outside** the lock — that is the sliding timeout.

### Locking pattern (important)

`subscription.removeSubscriber` deliberately fires `broker.tryRemoveSubscription` in a **separate goroutine**. The comment in `broker.go` explains: holding `subscription` write → `broker` write while another path can do `broker` → `subscription` would deadlock, so the removal is decoupled. If you touch the locking order, preserve this pattern; re-check for deadlocks with `go test -race`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [F2077/go-pubsub](https://github.com/F2077/go-pubsub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
