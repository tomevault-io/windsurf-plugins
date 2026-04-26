---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Commands

### Build & Test
```bash
make build              # Build cproxy and sproxy to bin/
make build-dev         # Build all binaries (including mockllm/mockagent) to release/
make release           # Cross-platform release packages to dist/
make test              # Run all tests
make test-race         # Run with race detector
make test-cover        # Generate coverage.html report
make test-pkg PKG=./internal/quota/...  # Test specific package
```

### Code Quality
```bash
make fmt               # Format with gofmt/goimports
make vet               # Run go vet
make lint              # Run golangci-lint
make tidy              # Update go.mod/go.sum
```

### Development
```bash
make run-sproxy        # Run sproxy with example config
make run-cproxy        # Run cproxy with example config
make bcrypt-hash       # Generate admin password hash
```

## Architecture Overview

**PairProxy** is an enterprise-grade LLM API gateway for Claude Code. Two-tier architecture:

```
Claude Code → cproxy (local:8080) → sproxy (server:9000) → Anthropic/OpenAI/Ollama
```

### Core Components

| Component | Purpose |
|-----------|---------|
| **cproxy** | Local proxy: intercepts requests, injects JWT, auto-refreshes tokens, load balances across sproxy instances |
| **sproxy** | Central gateway: JWT auth, quota management, token tracking, load balancing, protocol conversion, web dashboard, clustering |
| **internal/auth** | JWT issuance/refresh (24h access, 7d refresh), bcrypt hashing, LDAP/AD support, sk-pp- API Key generation (v2.15.0+) |
| **internal/quota** | Per-user/group daily/monthly limits, RPM rate limiting, sliding window algorithm, fail-open design |
| **internal/lb** | Weighted random balancer, active/passive health checks, circuit breaker, configurable retry (v2.17.0+) |
| **internal/db** | SQLite (default) or PostgreSQL, GORM ORM, async writes with buffering |
| **internal/proxy** | HTTP handlers, middleware, Anthropic ↔ OpenAI protocol conversion, streaming support |
| **internal/tap** | SSE stream parsing, zero-buffering token counting (input/output) |
| **internal/cluster** | Primary+Worker (SQLite) or Peer Mode (PostgreSQL, v2.14.0+) |
| **internal/router** | Semantic intent-based LLM target routing (v2.18.0+) |
| **internal/corpus** | Training data collection as JSONL (v2.16.0+) |
| **internal/track** | Full conversation recording per user |

### Key Features
- Zero-config client experience (set 2 env vars)
- Real-time token tracking via SSE parsing
- Multi-tenant with quotas, rate limiting, LDAP integration
- Protocol interoperability (Anthropic ↔ OpenAI auto-conversion)
- High availability (clustering, health checks, circuit breakers)
- Enterprise features (audit logs, metrics, webhooks, OpenTelemetry)
- Advanced routing (semantic intent-based, v2.18.0)
- Training data collection (corpus, v2.16.0)

## Code Style & Conventions

### Basics
- **Go version**: 1.24.0
- **Package names**: Lowercase, no underscores
- **Imports**: Standard library → external → internal
- **Comments**: Match existing file style (English or Chinese)

### Naming
- **Types**: PascalCase (`LLMTarget`, `Checker`)
- **Functions/Variables**: camelCase (`checkQuota`, `llmBalancer`)
- **Test functions**: `TestXxx` pattern
- **Error variables**: `ErrXxx` (`ErrNoLLMBinding`)

### Error Handling
- Wrap with context: `fmt.Errorf("context: %w", err)`
- Never ignore errors (add `//nolint:errcheck` with comment if intentional)
- **Fail-open**: Quota/database errors should not block requests (log and bypass)

### Logging (Zap)
- Create package logger: `logger.Named("subsystem")`
- **DEBUG**: Per-request details (disabled in prod)
- **INFO**: Lifecycle events
- **WARN**: Recoverable errors (DB write failure, health check failure)
- **ERROR**: Non-recoverable errors requiring manual intervention
- Always add context: `zap.String("user_id", id), zap.Error(err)`

## Testing Requirements

### Unit Tests
```bash
go test ./...                    # All tests
go test ./internal/quota/...     # Specific package
go test -cover ./...             # Coverage
```

### E2E Tests (3 types required)

**1. httptest automation** (daily development, CI/CD):
```bash
go test ./test/e2e/...
```

**2. Real process integration** (complete chain):
```bash
go test -tags=integration ./test/e2e/...
```

**3. Manual end-to-end** (debugging, stress testing):
```bash
./mockllm.exe --addr :11434 &
./sproxy.exe start --config test-sproxy.yaml &
./cproxy.exe start --config test-cproxy.yaml &
echo -e "testuser\ntestpass123" | ./cproxy.exe login --server http://localhost:9000
./mockagent.exe --url http://localhost:8080 --count 100 --concurrency 10
```

### Standards
- All unit tests must pass
- E2E types 1 & 2 must pass before merge
- New features require corresponding tests
- Run `make test-race` before merging concurrent changes
- No external assert frameworks (use standard `testing` package)

### Test Design Rules (防回归 checklist)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/l17728) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
