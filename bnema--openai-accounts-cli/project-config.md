---
trigger: always_on
description: Agent guidance for working in this Go CLI repository.
---

# AGENTS.md — openai-accounts-cli (`oa`)

Agent guidance for working in this Go CLI repository.

---

## Build & Run

```bash
go build -v ./...                        # Build all packages
go build -o oa ./cmd/oa                  # Build the `oa` binary
go run ./cmd/oa [command]                # Run without building
go mod tidy                              # Sync dependencies
go generate ./internal/ports/...         # Regenerate mocks (requires mockery v3)
```

---

## Test Commands

```bash
go test ./...                            # Run all tests
go test -v ./...                         # Verbose output
go test -race ./...                      # With race detector

# Run a single named test
go test -run TestServiceSetAuthSuccess ./internal/application/
go test -run TestRepositoryRoundTrip ./internal/adapters/repo/toml/
go test -run TestUsageCommandFetchesLimitsAndRendersStatus ./cmd/

# Run all tests in a specific package
go test ./internal/domain/
go test ./internal/adapters/secrets/chain/
go test ./internal/e2e/                  # E2E: builds a real binary (slow)
```

The general pattern for a single test is:
```bash
go test -run <TestFunctionName> ./<package-path>/
```

---

## Lint & Format

```bash
go fmt ./...          # Required before every commit
go vet ./...          # Static analysis (implied by CI)
```

There is no `.golangci.yml`; only standard Go tooling is configured. Generated mock files use `goimports` (configured in `.mockery.yaml`).

---

## Architecture

This project follows **hexagonal (ports & adapters)** architecture. Respect the dependency rule strictly:

```
cmd/          → application, ports, adapters (CLI wiring only)
application/  → domain, ports (use-case orchestration, no adapters)
adapters/     → ports, domain (concrete implementations)
ports/        → domain (interface definitions only)
domain/       → nothing internal (pure business logic)
```

- `internal/domain/` — Entities, value objects, sentinel errors; zero external deps.
- `internal/ports/` — Interface definitions (contracts); no concrete code.
- `internal/ports/mocks/` — Generated mocks; never edit manually.
- `internal/application/` — Service orchestration; depends only on `domain` and `ports`.
- `internal/adapters/` — Concrete port implementations (TOML repo, secrets, TUI render).
- `cmd/` — Cobra commands + manual DI wiring in `wire.go`.

---

## Code Style

### Imports

Three-block grouping: stdlib → internal → third-party. Use aliases when package names collide:

```go
import (
    "context"
    "fmt"

    "github.com/bnema/openai-accounts-cli/internal/domain"
    tomlrepo "github.com/bnema/openai-accounts-cli/internal/adapters/repo/toml"
    chainstore "github.com/bnema/openai-accounts-cli/internal/adapters/secrets/chain"

    "github.com/spf13/cobra"
)
```

### Naming Conventions

- **Types**: PascalCase — `AccountID`, `AuthMethod`, `LimitWindowKind`
- **String type aliases**: `type AccountID string` — use typed aliases, never raw `string`
- **Constants**: PascalCase grouped blocks — `AuthMethodAPIKey`, `LimitWindowDaily`
- **Constructors**: `NewXxx(...)` pattern — `NewService(...)`, `NewRepository(...)`
- **Command constructors**: lowercase `new` prefix — `newAccountCmd(app *app)`
- **Test functions**: `TestSubjectMethodScenario` — `TestServiceSetAuthSuccess`
- **Unexported internal types**: lowercase — `app`, `browserLoginConfig`, `fileSchema`
- **Interface compliance guards**: always include — `var _ ports.AccountRepository = (*Repository)(nil)`

### Error Handling

- Wrap with `fmt.Errorf("context noun: %w", err)` — lowercase, no trailing period, context first.
- Define sentinel errors as package-level vars in `errors.go` or at the top of relevant files:
  ```go
  var ErrAccountNotFound = errors.New("account not found")
  ```
- Use `errors.Is` for sentinel checks, `errors.As` for type assertions.
- Use `errors.Join` for multi-error aggregation (Go 1.20+).
- Unexported sentinels for internal-only errors: `var errNotImplementedYet = errors.New(...)`

### Structs and Types

- Pointer receivers for stateful types: `func (r *Repository) Get(...)`.
- Value receivers for pure logic: `func (u Usage) BlendedTotal() float64`.
- Nilable pointers for optional fields: `*Subscription`, `*AccountLimitSnapshot`.
- Struct tags: only in adapters/serialization layers (`toml:"..."`, `json:"..."`), never in `domain/`.

### Context

Every repository and store method accepts `context.Context` as its first argument. Check cancellation at the top:

```go
if err := ctx.Err(); err != nil {
    return err
}
```

### Concurrency

- `sync.RWMutex` for shared state (repositories, file stores).
- Per-path mutex map pattern (`lockForPath`) using a global map protected by a separate mutex.
- `sync.Once` for idempotent one-time operations.
- Bounded goroutines with a semaphore channel and `sync.WaitGroup` for concurrent HTTP work.

---

## Testing Patterns

- **Mocks**: Generated by mockery v3 in `internal/ports/mocks/`. Use `EXPECT()` builder syntax:
  ```go
  mockRepo.EXPECT().GetAccount(mock.Anything, accountID).Return(account, nil)
  ```
- **`require` vs `assert`**: Use `require` for fatal preconditions (stops test), `assert` for non-fatal checks.
- **Table-driven tests**: Preferred for multiple scenarios; use `t.Run(tt.name, ...)`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bnema) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
