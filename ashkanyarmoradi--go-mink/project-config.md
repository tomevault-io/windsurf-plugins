---
trigger: always_on
description: go-mink is an Event Sourcing and CQRS library for Go. Think of it as "MartenDB for Go" - making event sourcing as easy as using a traditional ORM.
---

# Copilot Instructions for go-mink

## Project Context

go-mink is an Event Sourcing and CQRS library for Go. Think of it as "MartenDB for Go" - making event sourcing as easy as using a traditional ORM.

## Key Concepts

### Event Sourcing
- Store all changes as immutable events, not current state
- Rebuild state by replaying events
- Events are facts that happened - never delete or modify them

### CQRS
- Separate write model (aggregates) from read model (projections)
- Commands change state, queries read state
- Optimize each side independently

## Architecture Quick Reference

```
Commands → Command Bus → Aggregate → Events → Event Store
                                                    ↓
                                           Projection Engine
                                                    ↓
                                             Read Models → Queries
```

## Core Interfaces

### EventStoreAdapter (adapters must implement)
```go
type EventStoreAdapter interface {
    Append(ctx context.Context, streamID string, events []EventRecord, expectedVersion int64) ([]StoredEvent, error)
    Load(ctx context.Context, streamID string, fromVersion int64) ([]StoredEvent, error)
    GetStreamInfo(ctx context.Context, streamID string) (*StreamInfo, error)
    GetLastPosition(ctx context.Context) (uint64, error)
    Initialize(ctx context.Context) error
    Close() error
}
```

### Aggregate (domain models implement)
```go
type Aggregate interface {
    AggregateID() string
    AggregateType() string
    Version() int64
    ApplyEvent(event interface{}) error
    UncommittedEvents() []interface{}
    ClearUncommittedEvents()
}
```

### Projection (read models implement)
```go
type Projection interface {
    Name() string
    HandledEvents() []string
    Apply(ctx context.Context, event StoredEvent) error
}
```

## Development Priorities

### Current Version: v1.0.0 (Stable)

All core features are complete: Event Store, Command Bus, Projection Engine, Saga Manager, Outbox Pattern, Event Versioning & Upcasting, Field-Level Encryption, GDPR Compliance, Observability, Testing Utilities, CLI Tool.

## Coding Standards

### Go Idioms
- Use `context.Context` as first parameter
- Return `error` as last return value
- Use `Option` function pattern for configuration
- Prefer `errors.Is()` over type assertions
- Use sentinel errors: `var ErrNotFound = errors.New("mink: not found")`

### Naming Conventions
- Interface single method: use `-er` suffix (`Serializer`, `Subscriber`)
- Constructors: `NewXxx()` prefix
- Test files: `xxx_test.go`
- Test functions: `TestXxx_Method_Scenario`

### Error Handling
```go
// Define sentinel errors
var ErrConcurrencyConflict = errors.New("mink: concurrency conflict")

// Use typed errors for details
type ConcurrencyError struct {
    StreamID        string
    ExpectedVersion int64
    ActualVersion   int64
}

// Implement Is() for errors.Is() compatibility
func (e *ConcurrencyError) Is(target error) bool {
    return target == ErrConcurrencyConflict
}
```

### Testing Patterns
```go
// Table-driven tests
func TestEventStore_Append(t *testing.T) {
    tests := []struct {
        name    string
        input   Input
        want    Output
        wantErr error
    }{...}
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {...})
    }
}

// BDD for aggregates (testing/bdd package)
bdd.Given(t, aggregate, previousEvents...).
    When(func() error { return aggregate.DoSomething() }).
    Then(expectedEvents...)
```

## Package Structure

```
mink/
├── *.go                 # Core types (root mink package), including:
│                        #   store.go, bus.go, projection_engine.go,
│                        #   saga_manager.go, outbox_processor.go,
│                        #   export.go (DataExporter), export_errors.go,
│                        #   encryption.go, versioning.go, errors.go
├── adapters/
│   ├── adapter.go       # All adapter interfaces and shared types
│   ├── postgres/        # PostgreSQL adapter (production)
│   └── memory/          # In-memory adapter (testing)
├── encryption/          # Provider interface, types, errors
│   ├── local/           # AES-256-GCM provider (testing/development)
│   ├── kms/             # AWS KMS provider (production)
│   └── vault/           # HashiCorp Vault Transit provider (production)
├── outbox/
│   ├── webhook/         # Webhook publisher
│   ├── kafka/           # Kafka publisher
│   └── sns/             # SNS publisher
├── middleware/
│   ├── metrics/         # Prometheus metrics
│   └── tracing/         # OpenTelemetry tracing
├── serializer/
│   ├── msgpack/         # MessagePack serializer
│   └── protobuf/        # Protocol Buffers serializer
├── testing/
│   ├── bdd/             # BDD test fixtures (Given-When-Then)
│   ├── assertions/      # Event assertions
│   ├── projections/     # Projection testing
│   ├── sagas/           # Saga testing
│   └── containers/      # PostgreSQL test containers
├── cli/commands/        # CLI tool
└── examples/            # Example projects
```

## PostgreSQL Schema

Events table uses JSONB for flexible event data:
```sql
CREATE TABLE mink_events (

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AshkanYarmoradi/go-mink](https://github.com/AshkanYarmoradi/go-mink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
