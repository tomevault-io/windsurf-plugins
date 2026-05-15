---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GuardianWAF is a zero-dependency Web Application Firewall written in Go (1.25+).
Module: `github.com/guardianwaf/guardianwaf`

The only Go dependency is `quic-go` (for optional HTTP/3 support, build with `-tags http3`).

## Key Constraints

- **ZERO external Go dependencies** — only Go stdlib (plus quic-go for HTTP/3). No exceptions.
- Frontend (React dashboard) uses npm packages — that's OK, they embed into the Go binary.
- Use `any` instead of `interface{}`
- Use built-in `min`/`max` functions (Go 1.21+)
- Use `range N` for simple loops (Go 1.22+)
- Use `slices.Contains` where applicable

## Build & Test

```bash
# Build and development
make build          # Build binary (includes React dashboard)
make run            # Build + run serve mode
make ui             # Build React dashboard only
make ui-dev         # Dashboard dev mode (hot reload on :5173, proxies API to :9443)

# Testing
make test           # Run all tests with -race
make vet            # Run go vet
make lint           # Run golangci-lint
make bench          # Run benchmarks
make fuzz           # Run fuzz tests (30s each)
make cover          # Generate coverage report
make smoke          # Build + run smoke tests
make docker-test    # Full Docker Compose integration test

# E2E tests (requires running server — defaults to http://localhost:9443)
make e2e            # Run Playwright tests (Chromium)
make e2e-headed     # Run with browser visible
make e2e-all        # Run all browsers
make e2e-list       # List available E2E tests
# Override defaults: E2E_BASE_URL=http://... E2E_API_KEY=... make e2e

# Running single tests
go test -race -v ./internal/layers/detection/sqli/... -run TestDetector
go test -race -v ./internal/engine/... -run TestPipeline

# Quick validation during development
go test -race -count=1 ./internal/layers/detection/...
go vet ./...

# Code formatting
make fmt            # Format with gofmt -s
make tidy           # Run go mod tidy
```

## Architecture

### Pipeline (core pattern)

All WAF processing flows through a **layer pipeline** (`internal/engine/pipeline.go`). Layers implement the `Layer` interface (`Name() + Process(ctx *RequestContext) LayerResult`) and are sorted by `Order` constant. The pipeline:

1. Iterates layers in order (lowest Order first)
2. Skips `Detector` layers if the path matches an exclusion
3. Layers read `ctx.TenantWAFConfig` directly for per-tenant config overrides (race-free, per-request)
4. Accumulates `Finding` scores via `ScoreAccumulator`
5. **Short-circuits on `ActionBlock`** — immediately returns without running remaining layers
6. `ActionChallenge` only applies if current action is `ActionPass` (block takes priority)

### Request Context

`engine.RequestContext` (`internal/engine/context.go`) carries all per-request state. It's pooled via `sync.Pool` for zero-allocation hot paths:
- Acquired via `AcquireContext()` — parses HTTP request, reads/decompresses body (gzip/deflate), extracts client IP (trusted proxy aware: X-Forwarded-For → X-Real-IP → RemoteAddr; only trusts proxy headers from configured `trusted_proxies` CIDRs)
- Released via `ReleaseContext()` — resets all fields, returns to pool
- Populates JA4 TLS fingerprint fields from custom TLS handler data
- Carries `TenantID` and `TenantWAFConfig` for multi-tenant isolation

### Layer Order Constants

Defined in `internal/engine/layer.go`. **29 layers** are registered in the main pipeline (serve mode). Library mode (`guardianwaf.go`) wires only 6 core layers: IP ACL, Rate Limit, Sanitizer, Detection, Bot Detection, Response.

**Registered (in pipeline):**

| Order | Layer | Description |
|-------|-------|-------------|
| 1 | SIEM | Passive event forwarding to SIEM systems (Splunk, ELK, ArcSight) |
| 75 | Cluster | HTTP gossip + leader election; distributes IP bans across nodes |
| 76 | WebSocket | WebSocket handshake validation, connection limits |
| 78 | gRPC | gRPC request validation, method allowlists, protobuf wire format + schema validation |
| 85 | Zero Trust | mTLS client verification, device attestation, session trust levels |
| 95 | Canary | Canary release routing (% traffic to canary upstream) |
| 100 | IP ACL | Radix tree CIDR matching, runtime add/remove, auto-ban |
| 125 | Threat Intel | IP/domain reputation feeds with LRU cache |
| 140 | Cache | Response caching (memory/Redis backend) |
| 145 | Replay | Request/response recording for testing |
| 150 | CORS | Origin validation, preflight caching |
| 150 | Custom Rules | Geo-aware rule engine with dashboard CRUD |
| 200 | Rate Limit | Token bucket per IP/path, auto-ban |
| 250 | ATO Protection | Brute force, credential stuffing, password spray, impossible travel |
| 275 | API Security | JWT validation (RS256/ES256/HS256), API key auth |
| 280 | API Validation | Request/response schema validation (YAML-defined schemas) |
| 285 | GraphQL | Query depth/complexity/introspection limits |
| 300 | Sanitizer | Normalize + validate requests |
| 310 | API Discovery | Passive API endpoint discovery, OpenAPI generation |
| 350 | CRS | OWASP ModSecurity Core Rule Set parser and executor |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GuardianWAF/GuardianWAF](https://github.com/GuardianWAF/GuardianWAF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
