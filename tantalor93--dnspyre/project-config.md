---
trigger: always_on
description: `dnspyre` is a command-line DNS benchmark tool written in Go. It stress-tests and measures the performance of DNS servers, supporting plain DNS (UDP/TCP), DoT (DNS over TLS), DoH (DNS over HTTPS), and DoQ (DNS over QUIC). The project is a fork/rewrite of [dnstrace](https://github.com/redsift/dnstrace).
---

# Copilot Instructions for dnspyre

## Project Overview

`dnspyre` is a command-line DNS benchmark tool written in Go. It stress-tests and measures the performance of DNS servers, supporting plain DNS (UDP/TCP), DoT (DNS over TLS), DoH (DNS over HTTPS), and DoQ (DNS over QUIC). The project is a fork/rewrite of [dnstrace](https://github.com/redsift/dnstrace).

**Module path:** `github.com/tantalor93/dnspyre/v3`

**Go version:** See `go.mod` (currently Go 1.25+)

---

## Repository Layout

```
.
├── main.go                     # Entry point – calls cmd.Execute()
├── cmd/
│   └── root.go                 # CLI flag definitions (kingpin) and Execute() function
├── pkg/
│   ├── dnsbench/               # Core benchmark logic and data types
│   │   ├── benchmark.go        # Benchmark struct, Run(), init(), worker goroutines
│   │   ├── defaults.go         # Default constant values (timeouts, counts, etc.)
│   │   ├── metrics.go          # ResultStats, Counters, Timings structs
│   │   ├── result.go           # Result aggregation helpers
│   │   ├── query_factory.go    # DNS query message construction
│   │   ├── request_logging.go  # Optional per-request logging
│   │   ├── system_nameserver.go / *_unix.go / *_windows.go  # OS nameserver detection
│   │   └── testdata/           # Embedded default-domains file
│   ├── reporter/               # Reporting and output formatting
│   │   ├── report.go           # PrintReport() – orchestrates all output
│   │   ├── stdreporter.go      # Human-readable stdout output
│   │   ├── jsonreporter.go     # JSON output mode
│   │   ├── plot.go             # Graph generation (svg/png/jpg) via gonum/plot
│   │   ├── merge.go            # Merges per-worker ResultStats into totals
│   │   └── durationutils.go    # Duration formatting helpers
│   └── printutils/             # Colored stderr/stdout helpers
├── data/                       # Sample domain list files (2-domains, 500-domains, etc.)
├── docs/                       # MkDocs documentation source (Markdown)
├── hack/                       # docker-compose + prometheus.yml for local dev
├── .circleci/config.yml        # CircleCI: tests + cross-platform builds
├── .github/workflows/          # GitHub Actions: lint, goreleaser-check, release, pages
├── .golangci.yml               # golangci-lint v2 config
├── .goreleaser.yaml            # GoReleaser release pipeline config
└── go.mod / go.sum
```

---

## Key Design Concepts

- **`Benchmark` struct** (`pkg/dnsbench/benchmark.go`) is the central data type. It holds all configuration and exposes a `Run(ctx context.Context) ([]*ResultStats, error)` method.
- **CLI → Benchmark**: `cmd/root.go` wires all CLI flags directly onto a package-level `Benchmark{}` value using kingpin. No separate config struct.
- **Concurrency model**: `Benchmark.Run()` spawns `Benchmark.Concurrency` worker goroutines. Each worker loops over the query list and sends DNS requests. Results are collected per-worker as `*ResultStats` slices.
- **Protocol dispatch**: The protocol used (UDP, TCP, DoT, DoH, DoQ) is determined during `Benchmark.init()` by inspecting the `Server` field and the `DOT`/`TCP` flags. The `useDoH` and `useQuic` booleans are set there.
- **Data sources** (`Benchmark.Queries`): Inline domain names, local files prefixed with `@`, or HTTP URLs. Loaded and de-duplicated in `Benchmark.init()`.
- **Reporting**: After `Run()` completes, `reporter.PrintReport()` merges per-worker stats and dispatches to the configured reporter (stdout table, JSON, CSV, plots).

---

## Development Commands

### Build
```bash
go build -v -o ./bin/dnspyre .
```

### Run tests (with race detector)
```bash
go test -race ./...
```

Tests use `gotestsum` in CI, but plain `go test` works locally.

### Lint
```bash
# Requires golangci-lint v2 (see .golangci.yml)
golangci-lint run
```

Linters enabled: `gocritic`, `godot`, `gosec`, `govet`, `ineffassign`, `revive`, `staticcheck`, `testifylint`, `unconvert`, `unparam`, `unused`, `usestdlibvars`, `wastedassign`, `whitespace`.

Formatters enforced: `gci`, `gofmt`, `gofumpt`, `goimports`.

### Check GoReleaser config
```bash
goreleaser check
```

---

## Testing Patterns

- Unit tests for `Benchmark.init()` live in `pkg/dnsbench/benchmark_test.go` (package `dnsbench` – white-box).
- Integration/API tests (`benchmark_*_api_test.go`) live in package `dnsbench_test` (black-box). They spin up a real in-process DNS server using `server_test.go`'s `NewServer()` helper (backed by `github.com/miekg/dns`).
- The `testify` library is used throughout (`assert` and `require`).
- Test helpers that produce DNS responses (e.g., returning A/AAAA records) are defined per protocol file (`benchmark_plaindns_api_test.go`, `benchmark_doh_api_test.go`, etc.).
- Reporter tests use golden-file comparison in `pkg/reporter/testdata/`.

**Adding a test:** Follow the existing table-driven test style (`tests []struct { name string; ... }`). For new protocol behaviour, add a `benchmark_<protocol>_api_test.go` file in `pkg/dnsbench/`.

---

## CI / Workflows


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tantalor93/dnspyre](https://github.com/Tantalor93/dnspyre) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
