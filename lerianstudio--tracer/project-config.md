---
trigger: always_on
description: Universal entry point for any AI coding agent working on the Tracer codebase.
---

# AGENTS.md — AI Agent Quick Reference

Universal entry point for any AI coding agent working on the Tracer codebase.

## Project Identity

**Tracer** is a real-time transaction validation and fraud prevention API built by Lerian Studio. It provides instant ALLOW/DENY/REVIEW decisions for financial transactions using CEL rule expressions, multi-scope spending limits, and an immutable audit trail with hash chain verification for SOX/GLBA compliance.

- **Language**: Go 1.25 (module `go 1.25.7`)
- **Architecture**: Hexagonal Architecture (Ports & Adapters) + CQRS
- **Database**: PostgreSQL 16
- **Rule Engine**: Google CEL (cel-go v0.27.0) with in-memory cache
- **Auth**: lib-auth v2 (API Key + Access Manager plugin)
- **License**: Elastic License 2.0

## Quick Start

```bash
cp .env.example .env   # Create environment file
make up                # Start PostgreSQL + Tracer
make test              # Run all tests
make lint              # golangci-lint v2
```

Health: `GET http://localhost:4020/health`

## Architecture Overview

Single service with four bounded contexts, all under `internal/`:

| Context | Role |
|---------|------|
| **Validation** | Orchestrate validation requests, coordinate rules + limits, record audit |
| **Rules** | Manage rule lifecycle (DRAFT→ACTIVE→INACTIVE→DELETED), compile/evaluate CEL |
| **Limits** | Spending limits (DAILY/MONTHLY/WEEKLY/PER_TRANSACTION/CUSTOM), usage counters |
| **Audit** | Immutable event log, hash chain verification, SOX/GLBA compliance |

### Layer Structure

```
internal/
├── bootstrap/         # Composition root: config, DI, server, workers
├── adapters/
│   ├── http/in/       # Fiber handlers, routes, middleware, validation
│   ├── postgres/      # Repository implementations (squirrel SQL builder)
│   └── cel/           # CEL expression engine adapter
├── services/          # Business logic
│   ├── command/       # Write operations (create, update, activate, deactivate, draft, delete)
│   ├── query/         # Read operations (get, list, evaluate, check limits, verify audit)
│   ├── cache/         # In-memory rule cache with warmup + background sync
│   └── workers/       # RuleSyncWorker, UsageCleanupWorker
├── testutil/          # Shared test helpers
pkg/
├── model/             # Domain entities (Rule, Limit, Validation, Scope, AuditEvent)
├── constant/          # Error codes (TRC-XXXX), pagination defaults
├── clock/             # Clock interface (Real + Fixed for MOCK_TIME)
└── resilience/        # Circuit breaker (sony/gobreaker wrapper)
```

## Essential Commands

| Command | Purpose |
|---------|---------|
| `make build` | Build binary to .bin/tracer |
| `make run` | Run locally with .env config |
| `make test` | All tests |
| `make test-unit` | Unit tests with race detector |
| `make test-integration` | Integration tests (testcontainers, -p=1) |
| `make test-e2e` | E2E BDD tests (Godog/Gherkin) |
| `make lint` | golangci-lint v2 with auto-fix |
| `make sec` | gosec + govulncheck |
| `make generate` | go generate (mocks) |
| `make generate-docs` | Regenerate Swagger docs |
| `make migrate` | Apply database migrations |
| `make up` / `make down` | Docker Compose lifecycle |

## Code Conventions

### Entity Constructors
```go
func NewRule(name, expression string, action Decision) (*Rule, error)
```
Validate all invariants. Return `(*T, error)`. Never panic. Defensive copies for slices.

### Service Methods
```go
func (s *ActivateRuleService) Execute(ctx context.Context, ruleID uuid.UUID) (*model.Rule, error) {
    logger, tracer, _, _ := libCommons.NewTrackingFromContext(ctx)
    ctx, span := tracer.Start(ctx, "service.rule.activate")
    defer span.End()
    logger = logging.WithTrace(ctx, logger)
    // ...
}
```
Always start with tracking + span. Enrich logger with trace context.

### Repository Pattern
- Interfaces defined where used (command/ and query/ packages)
- Separate PostgreSQL model structs from domain entities
- Squirrel SQL builder with `squirrel.Dollar` placeholder format
- Cursor-based pagination (no offset)

### Error Handling
- Sentinels: `var ErrRuleNotFound = errors.New("TRC-0100")` in `pkg/constant/errors.go`
- Wrapping: `fmt.Errorf("context: %w", err)` — always `%w`, never `%v`
- Business: `libOtel.HandleSpanBusinessErrorEvent(span, "msg", err)` — span stays OK
- Technical: `libOtel.HandleSpanError(span, "msg", err)` — span marked ERROR

### HTTP Responses
- Always use lib-commons wrappers: `libHTTP.OK()`, `libHTTP.Created()`, `libHTTP.WithError()`
- Never use direct Fiber responses (`c.JSON()`, `c.Status().JSON()`)

## Testing Requirements

| Tag | Scope | Run with |
|-----|-------|----------|
| (none) | Unit tests | `make test-unit` |
| `//go:build integration` | Testcontainers | `make test-integration` |
| `//go:build e2e` | Full stack BDD | `make test-e2e` |

- TDD required: write test first, then implement
- Table-driven tests with gomock (never manual mocks)
- Deterministic data: `testutil.FixedTime()`, `testutil.MustDeterministicUUID(seed)`
- Never use `uuid.New()` or `time.Now()` in tests
- `require.Len` before indexing slices
- No `defer ctrl.Finish()` — go.uber.org/mock auto-registers cleanup

## PR Standards

- Conventional commit format in PR titles

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LerianStudio/tracer](https://github.com/LerianStudio/tracer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
