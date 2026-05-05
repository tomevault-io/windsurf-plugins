---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md - Traceway Project

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Traceway is an error tracking and monitoring platform consisting of:
- **Frontend**: SvelteKit 2 dashboard application with Svelte 5
- **Backend**: Go/Gin API server with ClickHouse database
- **Go Client SDK**: Distributed tracing SDK for Go applications (external repo)

---

## Code Style

- **No pointless comments**: Do not add comments that simply describe what the code does. The code should be self-explanatory. Only add comments when explaining non-obvious "why" decisions.
- **No `py-4` in dialog form content**: Do not add `py-4` on the content wrapper inside `AlertDialog` or `Dialog` components — it creates too much blank space between the form and the action buttons.
- **Dialog button labels & toasts**: For form dialogs, use descriptive button labels with icons instead of generic "Create"/"Update". Create actions: `<Plus icon> {Action} {Entity}` (e.g., "+ New Widget Group"). Update actions: `<Check icon> Update {Entity}`. After successful create/update, show a `toast.success('Successfully {action} the {Entity}', { position: 'top-center' })`. The button should only be `disabled` during the loading state — never disable it to enforce validation; let the backend return 422 and show the error in the dialog instead.

---

## Quick Start

### Development Commands
| Component | Command | Description |
|-----------|---------|-------------|
| Frontend | `cd frontend && npm run dev` | Dev server (port 5173) |
| Frontend | `npm run build` | Production build |
| Frontend | `npm run check` | TypeScript checking |
| Backend | `cd backend && go run .` | API server (port 8082) |
| Go Client | External repo at `/Users/dusanstanojevic/Documents/workspace/go-client` | Build with `go build ./...` |

### Tech Stack
- **Frontend**: SvelteKit 2.49, Svelte 5.45, Tailwind CSS v4, shadcn-svelte, Vite 7
- **Backend**: Go 1.25, Gin 1.11, ClickHouse, PostgreSQL
- **Client SDK**: Go 1.25, Gin middleware support

### go-lightning Library (PostgreSQL ORM)
- **Import**: `github.com/tracewayapp/go-lightning/lit`
- **Purpose**: Lightweight generic CRUD operations for PostgreSQL

#### Model Registration (required before use)
All models are registered centrally in `models/models.go` via `models.Init()`:
```go
func Init() {
    lit.RegisterModel[User](lit.PostgreSQL)
    lit.RegisterModel[Project](lit.PostgreSQL)
    // ...all models registered here
}
```
Repository-local result models (e.g., aggregate structs only used in one repo) can use file-level `init()` instead.

#### Naming Conventions
- Fields: CamelCase → snake_case (`FirstName` → `first_name`)
- Consecutive uppercase: stay together (`HTTPCode` → `http_code`)
- Tables: pluralize + snake_case (`User` → `users`)
- Override via struct tag: `lit:"custom_name"`

#### Core CRUD Operations
All lit functions take `*sql.Tx` as the first argument for transactional consistency:

| Function | Description |
|----------|-------------|
| `lit.Insert[T](tx, &entity)` | Insert, returns auto-generated int ID |
| `lit.InsertUuid[T](tx, &entity)` | Insert with auto-generated UUID |
| `lit.InsertExistingUuid[T](tx, &entity)` | Insert with pre-set UUID |
| `lit.Select[T](tx, query, args...)` | Retrieve multiple records (returns `[]*T`) |
| `lit.SelectSingle[T](tx, query, args...)` | Retrieve one record (returns `*T`) |
| `lit.Update[T](tx, &entity, "id = $1", id)` | Update (auto-prepends WHERE) |
| `lit.UpdateNative(tx, "UPDATE table SET col = $1 WHERE ...", args...)` | Raw SQL update for partial/single-field changes |
| `lit.Delete(tx, "DELETE FROM table WHERE id = $1", id)` | Delete records |

#### Transaction Helper (`pgdb.ExecuteTransaction`)
All PostgreSQL operations should use `ExecuteTransaction` for automatic commit/rollback:

```go
// ExecuteTransaction[T] wraps a function in a transaction
// - Commits on success, rolls back on error or panic
// - Returns (T, error) directly - no pointer wrapping

project, err := pgdb.ExecuteTransaction(func(tx *sql.Tx) (*models.Project, error) {
    // All repository calls receive the transaction
    return repositories.ProjectRepository.FindById(tx, id)
})
```

#### Transactional Middleware (`middleware.Transactional`)
For auth flows and routes requiring transaction context throughout the request lifecycle, use the `Transactional` middleware:

```go
// In routes.go - wrap routes that need transaction context
api.POST("/register", middleware.Transactional, authController.Register)
api.POST("/login", middleware.Transactional, authController.Login)

// In controller - retrieve transaction from Gin context
func (c *AuthController) Register(ctx *gin.Context) {
    tx := middleware.GetTx(ctx)  // Get transaction from context

    // Use tx for all repository calls
    user, err := repositories.UserRepository.FindByEmail(tx, email)
    if err != nil {
        ctx.JSON(500, gin.H{"error": err.Error()})
        return  // Transaction auto-rolls back on non-success status
    }

    ctx.JSON(201, user)  // Transaction auto-commits on 200/201/303
}
```

**Auto-commit/rollback behavior:**
- Commits on status codes: 200, 201, 303

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tracewayapp/traceway](https://github.com/tracewayapp/traceway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
