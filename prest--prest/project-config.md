---
trigger: always_on
description: These instructions guide AI-generated changes in this repository.
---

# Copilot Instructions for pREST Go Backend

These instructions guide AI-generated changes in this repository.

## Project Scope

- Repository: `github.com/prest/prest`
- Language: Go (`go 1.26.0`, always check `go.mod`)
- Primary domain: PostgreSQL-backed REST API server, CLI, middleware, adapters, and plugins.
- Runtime entrypoint: `cmd/prestd/main.go`

## Core Engineering Principles

- Prefer small, focused changes over broad refactors.
- Preserve existing public behavior unless the task explicitly requires a change.
- Keep compatibility with Go 1.26.0.
- Favor readability and maintainability over clever abstractions.
- Do not introduce breaking changes to command flags, config keys, routes, or plugin behavior without explicit request.
- Apply SOLID when designing packages, ports, handlers, and adapters (see **SOLID Principles for Go** below).
- Keep the runtime stateless: no shared mutable application state between commands or requests (see **Go Style and Implementation Rules**).

## SOLID Principles for Go

Apply these on every change. They complement **Hexagonal Architecture** (below) — hexagonal defines layer boundaries; SOLID defines how types and dependencies inside those layers are shaped.

### S — Single Responsibility

One type or package should have one reason to change.

**pREST mapping:** `AuthHandler` handles authentication only; `CRUDHandler` handles table CRUD; `adapters/postgres/internal/connection` handles pooling only. Cross-cutting concerns (JWT, ACL, cache) belong in `middlewares/`, not inside handlers.

```go
// GOOD: focused handler
type AuthHandler struct {
    executor adapters.QueryExecutor
    cfg      AuthConfig
}

// BAD: god handler
type APIHandler struct {
    adapter adapters.Adapter // auth, CRUD, catalog, scripts...
}
```

### O — Open/Closed

Extend behavior via new types and interfaces, not by editing stable call sites.

**pREST mapping:** Add a new port in `adapters/` (e.g. `PermissionsChecker`), implement in `adapters/postgres/`, inject via `controllers.Deps`. Optional lifecycle (`DatabaseConnector`, `DatabasePinger`) stays outside composite `Adapter` so mocks stay minimal.

```go
// GOOD: extend via new port + wiring
type Deps struct {
    Perms adapters.PermissionsChecker // new concern → new field
}

// BAD: modify CRUDHandler for every new cross-cutting concern
func (h *CRUDHandler) handle(w http.ResponseWriter, r *http.Request) {
    if newFeatureFlag { /* special case */ }
}
```

### L — Liskov Substitution

Any implementation of a port must satisfy the full contract callers rely on.

**pREST mapping:** `adapters/mockgen/` and `adapters/postgres/` must be interchangeable for the ports handlers use. Test doubles must not panic or return inconsistent results on methods production code calls.

```go
// GOOD: mock implements the port handlers actually call
ctrl := gomock.NewController(t)
exec := mockgen.NewMockQueryExecutor(ctrl)
exec.EXPECT().QueryCtx(gomock.Any(), gomock.Any(), gomock.Any()).Return(scanner)

// BAD: partial stub that panics on unimplemented methods
type brokenExecutor struct{}
func (brokenExecutor) QueryCtx(...) adapters.Scanner { panic("not implemented") }
```

### I — Interface Segregation

Depend on the smallest interface that suffices; avoid fat interfaces at call sites.

**pREST mapping:** `NewAuthHandler(executor adapters.QueryExecutor, …)` needs only query execution. `NewTableHandler(executor, db, singleDB)` needs executor + registry, not full `Adapter`. Split ports live in `adapters/*.go`; composite `Adapter` is for the composition root only.

```go
// GOOD: narrow dependency
type TableHandler struct {
    executor adapters.QueryExecutor
    db       adapters.DatabaseRegistry
}

// BAD: forces mocks to implement 50+ methods
type TableHandler struct {
    adapter adapters.Adapter
}
```

### D — Dependency Inversion

High-level modules (`controllers/`, `middlewares/`) depend on abstractions (`adapters/*`), not concretions (`adapters/postgres`).

**pREST mapping:** `controllers.NewDepsFromConfig` maps `p.Adapter` into `Deps` port fields. Concrete postgres wiring happens in `cmd/prestd/` and `router/`, not inside handlers.

```go
// GOOD: depend on port, inject at edge
func NewCRUDHandler(deps Deps) *CRUDHandler {
    return &CRUDHandler{
        executor: deps.Executor, // adapters.QueryExecutor
    }
}

// BAD: construct concrete adapter inside handler
func NewCRUDHandler(cfg *config.Prest) *CRUDHandler {
    pg := postgres.New(cfg) // inverted dependency
}
```

### Quick reference

| Principle | pREST shorthand | Primary packages |
|-----------|-----------------|------------------|
| SRP | One handler/port per concern | `controllers/*`, `adapters/*.go` |
| OCP | New port + adapter, not `switch` hacks | `adapters/`, `controllers/deps.go` |
| LSP | mockgen and postgres interchangeable | `adapters/mockgen/`, tests |
| ISP | Smallest port on handler structs | `controllers/`, `adapters/` |
| DIP | No `adapters/postgres` outside wiring | `cmd/`, `router/`, `integration/` |

## Code Organization Conventions

- Keep new code in the most relevant existing package instead of creating new top-level folders.
- Follow existing package boundaries and hexagonal roles (see **Hexagonal Architecture** below).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prest/prest](https://github.com/prest/prest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
