---
trigger: always_on
description: 1. **Architecture** - New patterns, packages, dependencies
---

# Cloud Native Utils

## Documentation Policy

### Update CLAUDE.md when changes affect:
1. **Architecture** - New patterns, packages, dependencies
2. **API surface** - New interfaces, functions, types
3. **Conventions** - New naming rules, anti-patterns, gotchas
4. **Decisions** - Architectural trade-offs or technology choices

### Update README.md when changes affect:
1. **User-facing behavior** - New packages, usage examples
2. **Setup instructions** - New prerequisites, Go version requirements

### Documentation Checklist (before commit):
- [ ] New package → Add to Project Structure and README?
- [ ] New pattern/gotcha → Add to Conventions or Gotchas section?
- [ ] New interface → Document in Package Patterns?
- [ ] New environment variable → Add to `.env.example`?
- [ ] Roadmap item completed → Mark as [x]?

### Documentation Update Matrix

| Change Type | CLAUDE.md Section | README.md |
|-------------|-------------------|-----------|
| New package | Project Structure, Roadmap | Features, Usage |
| New interface | Package Patterns | Usage examples |
| New convention | Coding Conventions | - |
| New gotcha | Gotchas | - |
| Architectural decision | Decisions | - |
| New environment variable | - | .env.example |

---

## Project Structure

```
cloud-native-utils/
├── assert/          Test assertions (assert.That)
├── consistency/     Transactional event log (JSON file persistence)
├── efficiency/      Channel helpers, gzip middleware, similarity search, sparse data structures
├── env/             Generic environment variable parsing
├── event/           Domain event interfaces
├── extensibility/   Dynamic Go plugin loading
├── logging/         Structured JSON logging (log/slog)
├── mcp/             Model Context Protocol server (Claude Desktop)
├── messaging/       Pub-sub dispatchers (in-memory, Kafka)
├── resource/        Generic CRUD backends (memory, sharded-sparse, JSON, YAML, SQLite, PostgreSQL)
├── security/        AES-GCM encryption, password hashing, HMAC
├── service/         Context helpers, lifecycle management
├── slices/          Generic slice utilities
├── stability/       Resilience patterns (breaker, retry, throttle, debounce, timeout)
├── templating/      HTML template engine (embed.FS support)
└── web/             HTTP server, client, sessions, OIDC, auth middleware
```

### File Naming Conventions

| Pattern | Purpose |
|---------|---------|
| `{feature}.go` | Main implementation |
| `{feature}_test.go` | Tests for feature |
| `{package}.go` | Package-level documentation |
| `access.go` | Interface definitions |
| `testdata/` | Test fixtures and data |

---

## Commands

```
just test            Run all tests with coverage
just test-integration Run integration tests (requires tags)
just benchmark       Run consistency benchmarks
just lint            Run golangci-lint
just plugin          Build test plugins
just make-certs      Generate mTLS certificates
```

---

## Coding Conventions

### Generic Type Patterns

```go
// CRUD access with comparable key and any value
type Access[K comparable, V any] interface { ... }

// Implementation with generics
type InMemoryAccess[K comparable, V any] struct { ... }
```

### Function Signature Convention

```go
// Cloud-native service function pattern
type Function[IN, OUT any] func(ctx context.Context, in IN) (out OUT, err error)
```

### Constructor Naming

```go
// Always use New{Type} pattern
func NewInMemoryAccess[K comparable, V any]() *InMemoryAccess[K, V]
func NewServer(handler http.Handler) *Server
func NewJsonLogger() *slog.Logger
```

### Error Handling

```go
// Use named error constants
var (
    ErrorResourceAlreadyExists = errors.New("resource already exists")
    ErrorResourceNotFound      = errors.New("resource not found")
)

// Prefix with Err for sentinel errors
var ErrBreakerServiceUnavailable = errors.New("service unavailable")
```

### Context Propagation

- All operations require `context.Context` as first parameter
- Check context cancellation early in operations
- Propagate context through all function calls

### Concurrency Safety

```go
// Use sync.RWMutex for shared state
type InMemoryAccess[K comparable, V any] struct {
    data map[K]V
    mu   sync.RWMutex
}

// Read lock for read operations
func (a *InMemoryAccess[K, V]) Read(ctx context.Context, key K) (*V, error) {
    a.mu.RLock()
    defer a.mu.RUnlock()
    // ...
}

// Write lock for mutations
func (a *InMemoryAccess[K, V]) Create(ctx context.Context, key K, value V) error {
    a.mu.Lock()
    defer a.mu.Unlock()
    // ...
}
```

---

## Testing Conventions

### AAA Pattern

```go
func Test_Feature_With_Condition_Should_Outcome(t *testing.T) {
    // Arrange
    access := resource.NewInMemoryAccess[string, User]()
    ctx := context.Background()
    user := User{Name: "Alice"}

    // Act
    err := access.Create(ctx, "user-1", user)

    // Assert
    assert.That(t, "error should be nil", err, nil)
}
```

### Test Naming

```
Test_{Feature}_With_{Condition}_Should_{Outcome}

Examples:
- Test_Breaker_With_ThresholdExceeded_Should_ReturnServiceUnavailable
- Test_InMemoryAccess_With_DuplicateKey_Should_ReturnError
- Test_Retry_With_ContextCanceled_Should_ReturnContextError
```

### Assertion Usage

```go
// Use assert.That for all assertions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andygeiss/cloud-native-utils](https://github.com/andygeiss/cloud-native-utils) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
