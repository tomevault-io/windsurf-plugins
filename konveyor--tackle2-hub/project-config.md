---
trigger: always_on
description: This document defines the Go coding standards and patterns for the Tackle2 Hub project. Follow these conventions
---

# Tackle2 Hub Coding Patterns Conventions and Standards

This document defines the Go coding standards and patterns for the Tackle2 Hub project. Follow these conventions
to maintain consistency and code quality across the codebase.

## Table of Contents

- [Project Organization](#project-organization)
- [Package Organization](#package-organization)
- [Object-Oriented Design](#object-oriented-design)
- [Function Design](#function-design)
- [Function and Method Standards](#function-and-method-standards)
  - [Naming Conventions](#naming-conventions)
  - [Function Returns](#function-returns)
  - [Code Documentation and Comments](#code-documentation-and-comments)
- [Testing](#testing)
- [Error Handling](#error-handling)
- [Git Commits](#git-commits)

---

## Project Organization

### Quick Decision Guide

When creating a new type, ask:

1. **Will external clients/addons need this?** → `shared/`
2. **Is this a REST API request/response type?** → `shared/api/`
3. **Is this a database model?** → `internal/model/`
4. **Is this handler logic?** → `internal/api/`
5. **Does it import anything from `internal/`?** → Must stay in `internal/`

### Exported Types Go in shared/

**All exported types, interfaces, and APIs that are consumed by external packages or clients must live in
`shared/`.** This includes:

- REST API request/response types
- Client binding interfaces and implementations
- Public configuration structures
- Addon framework types and interfaces
- Any type that external code needs to import

```go
// Good - exported types in shared/
// File: shared/api/application.go
package api

// Application REST resource - exported for client use
type Application struct {
    Resource        `yaml:",inline"`
    Name            string        `json:"name" binding:"required"`
    Description     string        `json:"description"`
    BusinessService *Ref          `json:"businessService"`
    Owner           *Ref          `json:"owner"`
    Tags            []TagRef      `json:"tags"`
    // ... other exported fields
}

// File: shared/binding/application.go
package binding

// Application client - exported for external consumers
type Application struct {
    client *Client
}

func (h *Application) Create(r *api.Application) (err error) {
    // Client implementation
    return
}
```

```go
// Bad - putting exported client types in internal/
// File: internal/api/client.go
package api

// Don't export client-facing types from internal/
type ApplicationClient struct {
    // ...
}
```

### Internal Implementation Details

**Implementation details, handler logic, and hub-specific code stays in `internal/`.** This includes:

- HTTP handler implementations
- Database models and migrations
- Business logic and domain services
- Internal utilities and helpers

```go
// Good - internal handler implementation
// File: internal/api/application.go
package api

import (
    "github.com/konveyor/tackle2-hub/internal/model"
    "github.com/konveyor/tackle2-hub/shared/api"  // Import from shared OK
)

// ApplicationHandler - internal HTTP handler
type ApplicationHandler struct {
    BaseHandler
}

func (h ApplicationHandler) Get(ctx *gin.Context) {
    m := &model.Application{}  // Internal model
    // ... handler implementation
    r := api.Application{}     // Shared API type for response
    h.Respond(ctx, http.StatusOK, r)
}
```

### Critical Rule: shared/ Cannot Import from internal/

**NEVER import anything from `internal/` in `shared/` packages.** This is a hard architectural boundary that
ensures:
- Clean separation of concerns
- Clients don't accidentally depend on internal implementation details
- Internal refactoring doesn't break external consumers

```go
// FORBIDDEN - shared/ importing from internal/
// File: shared/api/application.go
package api

import (
    "github.com/konveyor/tackle2-hub/internal/model"  // ❌ NEVER DO THIS
)

type Application struct {
    Model model.Application  // ❌ Exposing internal types
}
```

```go
// Correct - shared/ is independent
// File: shared/api/application.go
package api

type Application struct {
    Name            string   `json:"name"`
    BusinessService *Ref     `json:"businessService"`
    // Only reference other shared/ types
}
```

### Import Direction

The allowed import direction is:

```
internal/  →  shared/     ✅ OK: Internal can import shared types
shared/    →  internal/   ❌ FORBIDDEN: Shared cannot import internal
external   →  shared/     ✅ OK: External clients import shared types
external   →  internal/   ❌ FORBIDDEN: Enforced by Go visibility rules
```

### Example: Correct Usage

```go
// ✅ Internal handler uses shared API types
// File: internal/api/application.go
package api

import (
    "github.com/konveyor/tackle2-hub/internal/model"
    "github.com/konveyor/tackle2-hub/shared/api"
)

func (h ApplicationHandler) Create(ctx *gin.Context) {
    r := &api.Application{}  // Shared API type
    err := h.Bind(ctx, r)
    m := r.Model()           // Convert to internal model
    // ... create in database
}
```

```go
// ✅ Shared binding has no internal dependencies
// File: shared/binding/application.go
package binding

import "github.com/konveyor/tackle2-hub/shared/api"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/konveyor) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
