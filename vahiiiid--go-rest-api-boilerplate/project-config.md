---
trigger: always_on
description: **Last Updated**: 2025-12-10
---

# GitHub Copilot Instructions for GRAB (Go REST API Boilerplate)

**Version**: v2.0.0  
**Last Updated**: 2025-12-10  
**Purpose**: Developer-focused guidelines for building APIs with GRAB

---

## 📋 What is GRAB?

GRAB (Go REST API Boilerplate) is a production-ready Go REST API starter with:
- **Clean Architecture** (Handler → Service → Repository)
- **JWT Authentication** with refresh token rotation
- **Role-Based Access Control (RBAC)**
- **Database Migrations** (golang-migrate)
- **Docker-First Development**
- **89.81% Test Coverage**
- **Comprehensive Documentation**: https://vahiiiid.github.io/go-rest-api-docs/

---

## 🎯 Core Development Principles

### 1. **Environment Detection - Don't Hardcode Versions**
Instead of stating "Go 1.24" or "PostgreSQL 15", show how to check:

```bash
# Check Go version
go version

# Check Docker version
docker --version

# Check PostgreSQL version (inside container)
make exec-db
psql --version
```

### 2. **Docker-First Development**
- Developers run `make` commands on host
- **Makefile automatically detects** if Docker container is running
- Commands execute in container if available, host otherwise
- **No need to manually enter container** - the Makefile handles execution context

```bash
# Start containers first
make up

# Run tests (automatically in container if running)
make test

# Run linting (automatically in container if running)
make lint

# Apply lint fixes (automatically in container if running)
make lint-fix

# Generate Swagger docs (automatically in container if running)
make swag
```

### 3. **Clean Architecture Pattern**
Every domain follows this structure:

```
internal/
└── <domain>/
    ├── model.go       # Domain models (GORM)
    ├── dto.go         # Data Transfer Objects (API contracts)
    ├── repository.go  # Database access layer
    ├── service.go     # Business logic layer
    ├── handler.go     # HTTP handlers (Gin)
    └── *_test.go      # Tests for each layer
```

**Key Rules**:
- Handler → Service → Repository (never skip layers)
- No business logic in handlers
- No HTTP concerns in services
- Repository only talks to database

---

## 🚀 Common Development Tasks

### Adding a New Domain/Entity

**Example**: Adding a "Todo" entity

1. **Create directory structure**:
```bash
mkdir -p internal/todo
```

2. **Create model** (`internal/todo/model.go`):
```go
package todo

import (
    "time"
    "gorm.io/gorm"
)

type Todo struct {
    ID          uint           `gorm:"primarykey" json:"id"`
    Title       string         `gorm:"not null" json:"title"`
    Description string         `json:"description"`
    Completed   bool           `gorm:"default:false" json:"completed"`
    UserID      uint           `gorm:"not null" json:"user_id"`
    CreatedAt   time.Time      `json:"created_at"`
    UpdatedAt   time.Time      `json:"updated_at"`
    DeletedAt   gorm.DeletedAt `gorm:"index" json:"-"`
}
```

3. **Create DTO** (`internal/todo/dto.go`):
```go
package todo

type CreateTodoRequest struct {
    Title       string `json:"title" binding:"required,min=3,max=200"`
    Description string `json:"description" binding:"max=1000"`
}

type UpdateTodoRequest struct {
    Title       string `json:"title" binding:"omitempty,min=3,max=200"`
    Description string `json:"description" binding:"omitempty,max=1000"`
    Completed   *bool  `json:"completed"`
}

type TodoResponse struct {
    ID          uint      `json:"id"`
    Title       string    `json:"title"`
    Description string    `json:"description"`
    Completed   bool      `json:"completed"`
    UserID      uint      `json:"user_id"`
    CreatedAt   time.Time `json:"created_at"`
    UpdatedAt   time.Time `json:"updated_at"`
}
```

4. **Create repository** (`internal/todo/repository.go`):
```go
package todo

import (
    "context"
    "gorm.io/gorm"
)

type Repository interface {
    Create(ctx context.Context, todo *Todo) error
    FindByID(ctx context.Context, id uint) (*Todo, error)
    FindByUserID(ctx context.Context, userID uint) ([]Todo, error)
    Update(ctx context.Context, todo *Todo) error
    Delete(ctx context.Context, id uint) error
}

type repository struct {
    db *gorm.DB
}

func NewRepository(db *gorm.DB) Repository {
    return &repository{db: db}
}

// Implementation methods...
```

5. **Create service** (`internal/todo/service.go`):
```go
package todo

import (
    "context"
    "go-rest-api-boilerplate/internal/errors"
)

type Service interface {
    CreateTodo(ctx context.Context, userID uint, req *CreateTodoRequest) (*TodoResponse, error)
    GetTodo(ctx context.Context, userID, todoID uint) (*TodoResponse, error)
    GetUserTodos(ctx context.Context, userID uint) ([]TodoResponse, error)
    UpdateTodo(ctx context.Context, userID, todoID uint, req *UpdateTodoRequest) (*TodoResponse, error)
    DeleteTodo(ctx context.Context, userID, todoID uint) error
}

type service struct {
    repo Repository
}

func NewService(repo Repository) Service {
    return &service{repo: repo}
}

// Implementation methods...
```

6. **Create handler** (`internal/todo/handler.go`):
```go
package todo

import (
    "net/http"
    "strconv"
    
    "github.com/gin-gonic/gin"
    "github.com/vahiiiid/go-rest-api-boilerplate/internal/contextutil"
    apiErrors "github.com/vahiiiid/go-rest-api-boilerplate/internal/errors"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vahiiiid/go-rest-api-boilerplate](https://github.com/vahiiiid/go-rest-api-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
