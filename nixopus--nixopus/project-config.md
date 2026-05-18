---
trigger: always_on
description: Go API Development Rules for Nixopus Backend
---


# Nixopus Go API Development Guidelines

You are a senior backend engineer building the Nixopus API — a production-grade Go application using the Fuego framework, Bun ORM, and domain-driven architecture. Your focus is on writing clean, maintainable, and extensible code following established patterns.

## Core Principles

### DRY (Don't Repeat Yourself) — Highest Priority
- **Before writing new code**, search the codebase for existing implementations
- Check `internal/utils/` for common utilities (`GetUser`, `SendErrorResponse`, `SendJSONResponse`)
- Check `internal/types/` for shared type definitions
- Reuse existing storage patterns and repository interfaces
- Extract common validation logic to shared validators
- Use existing middleware from `internal/middleware/`

### Single Responsibility Principle (SRP)
- **Controllers**: HTTP request/response handling only
- **Services**: Business logic and orchestration
- **Storage**: Database operations only (no business logic)
- **Validation**: Request validation only
- **Types**: Data structures and domain errors
- Each file should have one primary purpose

### Code Readability
```go
// ✅ Good: Early returns, flat structure
func (c *Controller) HandleRequest(f fuego.ContextNoBody) (*types.Response, error) {
    user := utils.GetUser(f.Response(), f.Request())
    if user == nil {
        return nil, fuego.UnauthorizedError{Detail: "authentication required"}
    }

    data, err := c.service.GetData(user.ID.String())
    if err != nil {
        c.logger.Log(logger.Error, err.Error(), "")
        return nil, fuego.HTTPError{Err: err, Detail: err.Error(), Status: http.StatusInternalServerError}
    }

    return &types.Response{
        Status:  "success",
        Message: "Data fetched successfully",
        Data:    data,
    }, nil
}

// ❌ Bad: Nested conditions
func (c *Controller) HandleRequest(f fuego.ContextNoBody) (*types.Response, error) {
    user := utils.GetUser(f.Response(), f.Request())
    if user != nil {
        data, err := c.service.GetData(user.ID.String())
        if err == nil {
            return &types.Response{Status: "success", Data: data}, nil
        } else {
            return nil, fuego.HTTPError{Err: err, Detail: err.Error(), Status: http.StatusInternalServerError}
        }
    }
    return nil, fuego.UnauthorizedError{Detail: "authentication required"}
}
```

## Architecture

### Directory Structure
```
api/
├── internal/
│   ├── features/              # Domain features
│   │   └── [domain]/
│   │       ├── controller/    # HTTP handlers
│   │       │   ├── init.go    # Controller struct & constructor
│   │       │   └── [action].go
│   │       ├── service/       # Business logic
│   │       │   ├── init.go    # Service struct & constructor
│   │       │   └── [action].go
│   │       ├── storage/       # Database operations
│   │       │   └── init.go    # Repository interface & implementation
│   │       ├── types/         # Domain-specific types & errors
│   │       │   └── init.go
│   │       ├── validation/    # Request validators
│   │       │   └── validator.go
│   │       └── tests/         # Unit tests
│   ├── middleware/            # HTTP middleware
│   ├── routes/                # Route registration
│   ├── storage/               # Shared storage (App, Store)
│   ├── types/                 # Shared types
│   └── utils/                 # Shared utilities
├── migrations/                # SQL migrations by domain
└── templates/                 # YAML templates
```

### Creating a New Feature Domain

1. Create the directory structure:
```
internal/features/[domain]/
├── controller/init.go
├── service/init.go
├── storage/init.go
├── types/init.go
├── validation/validator.go
└── tests/
```

2. Register routes in `internal/routes/[domain].go`
3. Add middleware configuration in `internal/routes/routes.go`

## Fuego Framework Patterns

### Controller Structure
```go
package controller

import (
    "context"
    "net/http"

    "github.com/nixopus/nixopus/api/internal/features/[domain]/service"
    "github.com/nixopus/nixopus/api/internal/features/[domain]/storage"
    "github.com/nixopus/nixopus/api/internal/features/[domain]/validation"
    "github.com/nixopus/nixopus/api/internal/features/logger"
    "github.com/nixopus/nixopus/api/internal/features/notification"
    shared_storage "github.com/nixopus/nixopus/api/internal/storage"
    shared_types "github.com/nixopus/nixopus/api/internal/types"
)

type DomainController struct {
    store        *shared_storage.Store
    validator    *validation.Validator
    service      *service.DomainService
    ctx          context.Context
    logger       logger.Logger
    notification *notification.NotificationManager
}

func NewDomainController(
    store *shared_storage.Store,
    ctx context.Context,
    l logger.Logger,
    notificationManager *notification.NotificationManager,
) *DomainController {
    storage := storage.DomainStorage{DB: store.DB, Ctx: ctx}
    return &DomainController{
        store:        store,
        validator:    validation.NewValidator(&storage),
        service:      service.NewDomainService(store, ctx, l, &storage),
        ctx:          ctx,
        logger:       l,
        notification: notificationManager,
    }
}
```

### Handler Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nixopus/nixopus](https://github.com/nixopus/nixopus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
