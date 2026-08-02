---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

prometheus-nats-exporter is a Prometheus exporter for NATS server monitoring. It aggregates metrics from NATS server HTTP monitoring endpoints (varz, connz, subz, routez, healthz, jsz, etc.) and exposes them in Prometheus format.

### Place in the ecosystem

- **Sibling project**: `nats-io/nats-surveyor` monitors NATS too, but via the NATS system account (`$SYS.REQ` messages, JSAPI advisories). This exporter is HTTP-poll only. Do **not** propose surveyor-style features (system account auth, advisory subscriptions) here — they belong in surveyor.
- **Deployment model**: One exporter sidecar per nats-server. Wired into `nats-io/k8s` Helm charts as a StatefulSet sidecar.
- **Grafana dashboards**: Pre-built JSON in `walkthrough/` (both `gnatsd`-prefixed and Helm `nats`-prefixed variants).

## Build and Test Commands

### Building
```bash
make build          # Build the binary
```

### Testing
```bash
make test           # Run all tests with race detection
make test-cov       # Run tests with coverage output (generates collector.out and exporter.out)

# Run tests for specific packages
go test -v -race -count=1 -parallel=1 ./test/...
go test -v -race -count=1 -parallel=1 ./collector/...
go test -v -race -count=1 -parallel=1 ./exporter/...
```

### Linting
```bash
make lint           # Run go vet and golangci-lint
```

Note: Linting requires golangci-lint to be installed. See `.golangci.yaml` for enabled linters.

### Formatting
Always format Go code with:
```bash
go fmt ./...
```

### Docker
```bash
make dockerx        # Build docker image using buildx
```

### Running the Exporter
```bash
# Basic usage - monitor varz endpoint
./prometheus-nats-exporter -varz "http://localhost:8222"

# With multiple endpoints
./prometheus-nats-exporter -varz -connz -routez "http://localhost:8222"

# Default metrics endpoint
# http://0.0.0.0:7777/metrics
```

If no metric flags are passed, the exporter defaults to `-varz` only (not "nothing"). See `main.go` flag handling.

## Architecture

### Package Structure

**`main.go`** - Entry point
- Parses command-line flags
- Creates exporter instance
- Manages graceful shutdown
- Supports multiple NATS servers (though not recommended per Prometheus best practices)
- Handles server ID parsing: can use URL-based ID, custom ID, or fetch from /varz endpoint

**`exporter/`** - HTTP server and coordination
- `NATSExporter`: Main coordinator that manages HTTP server and collectors
- Registers Prometheus collectors for each enabled metric type
- Handles basic auth for scrape endpoint if configured
- Supports TLS configuration
- Default listen: `0.0.0.0:7777/metrics`

**`collector/`** - Metric collection logic
- Each NATS endpoint has its own collector file (e.g., `varz.go`, `connz.go`, `jsz.go`)
- `NATSCollector`: Base type for standard collectors using reflection-based metric extraction (`objectToMetrics` in `collector.go`)
- JetStream collector (`jsz.go`): Custom implementation using Prometheus `Describe`/`Collect` pattern — required because nested account→stream→consumer hierarchy needs typed labels with fixed cardinality
- Custom (non-reflection) collectors: `connz.go`, `gatewayz.go`, `leafz.go`, `accountz.go`, `accstatz.go`, `healthz.go`, `jsz.go`. Reflection-only: `varz.go`, `subz.go`, `routez.go`
- Collectors poll NATS HTTP endpoints and transform JSON responses into Prometheus metrics
- All metrics are gauges (snapshots of current state)
- Metric naming: Uses "gnatsd" namespace for backward compatibility (not "nats")
- **Reflection skip-list** (`collector.go:338-348`): 8 top-level JSON fields are explicitly dropped (`cluster_*`, `gateway_*`, `trusted_operators_claim`). Adding a new field to varz/etc. does NOT automatically expose it if it matches one of these prefixes
- **`healthz.go` dual-metric pattern**: emits both numeric `status` (0=ok, 1=error) and `statusValue` with a `value` label (`ok|unreachable|error`). Both are intentional — preserve when modifying
- **`mapKeys()` schema drift**: collector tracks response keys across polls; if NATS server changes its JSON shape, collectors are re-initialized rather than silently dropping new fields

**`test/`** - Testing utilities
- Provides helpers to run embedded NATS servers for testing
- Default ports: ClientPort=11224, MonitorPort=11424, StaticPort=11425
- **Static fixture servers** for endpoints not easily exercised from an embedded NATS: `RunJszStaticServer`, `RunGatewayzStaticServer`, `RunAccstatzStaticServer`, `RunLeafzStaticServer`. New collector tests for these endpoints should use the static-server pattern, not try to coerce embedded NATS into producing the shape

### Key Architectural Patterns

1. **Dual API**: Can be used as standalone binary OR embedded as a Go library
2. **Collector Registration**: Each metric type (varz, connz, etc.) registers as a Prometheus collector
3. **Server ID Handling**: Three modes:
   - URL-based (default): Uses scheme://host as ID
   - Custom: Format `id,url`
   - Internal: Fetches ServerID or ServerName from /varz endpoint

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nats-io/prometheus-nats-exporter](https://github.com/nats-io/prometheus-nats-exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
