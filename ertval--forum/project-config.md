---
trigger: always_on
description: **Architecture**: Modular Monolith with Hexagonal Architecture (Ports & Adapters)
---

# Forum - AI Coding Agent Instructions

## Quick Reference

**Architecture**: Modular Monolith with Hexagonal Architecture (Ports & Adapters)  
**Language**: Go 1.24+ with minimal dependencies (SQLite driver, bcrypt, UUID only)  
**Status**: 75% Complete - Auth, Posts, Categories fully implemented with tests  
**Entry Point**: `cmd/forum/main.go` → `cmd/forum/wire/` for all DI wiring  
**Critical Files**: `docs/ARCHITECTURE.md`, `docs/IMPLEMENTATION_ROADMAP.md`, `docs/UNIFIED_DI_PATTERN.md`

---

## Architecture Pattern: The Hexagon

Each module follows **strict 4-directory structure**:

```
internal/modules/{module}/
├── domain/          # Pure business logic (stdlib only)
│   ├── {entity}.go  # Domain entities with Validate()
│   └── errors.go    # Domain-specific errors (errors.New())
├── ports/           # Interface definitions (contracts)
│   ├── service.go   # INPUT PORT - Use case interface
│   └── repository.go # OUTPUT PORT - Data access interface
├── application/     # Business logic orchestration
│   └── service.go   # Implements ports interfaces
└── adapters/        # Technical implementations (FLAT - no subdirs)
    ├── http_handler.go       # INPUT ADAPTER - HTTP handlers
    └── sqlite_repository.go  # OUTPUT ADAPTER - Database access
```

### Port/Adapter File Headers (MANDATORY)

Every file in `ports/` and `adapters/` MUST start with:
- `// INPUT PORT - Service Interface`
- `// OUTPUT PORT - Repository Interface`
- `// INPUT ADAPTER - HTTP Handler`
- `// OUTPUT ADAPTER - SQLite Repository`

**Example**: `internal/modules/auth/ports/service.go` starts with `// INPUT PORT - Service Interface`

### Dependency Rules (Enforced)

```
domain      → stdlib ONLY (no imports from project)
ports       → domain only
application → domain, ports
adapters    → domain, ports (never application!)
```

Cross-module: Import `ports.XService` interface ONLY. Never import `application/` or `adapters/` from other modules.

---

## Critical Workflow: Adding Features (Follow Exactly)

### 1. Domain Layer First
```go
// internal/modules/{module}/domain/{entity}.go
type Post struct {
    ID        string
    Title     string
    Content   string
    CreatedAt time.Time
}

func (p *Post) Validate() error {
    if p.Title == "" { return domain.ErrEmptyTitle }
    return nil
}

// internal/modules/{module}/domain/errors.go
var ErrEmptyTitle = errors.New("title cannot be empty")
```

### 2. Define Ports (Interfaces)
```go
// internal/modules/{module}/ports/service.go (INPUT PORT)
type PostService interface {
    CreatePost(ctx context.Context, userID, title, content string) (*domain.Post, error)
}

// internal/modules/{module}/ports/repository.go (OUTPUT PORT)
type PostRepository interface {
    Create(ctx context.Context, post *domain.Post) error
}
```

### 3. Implement Application Service
```go
// internal/modules/{module}/application/service.go
type Service struct {
    repo ports.PostRepository
}

func (s *Service) CreatePost(ctx context.Context, userID, title, content string) (*domain.Post, error) {
    post := &domain.Post{ID: generateID(), Title: title, Content: content}
    if err := post.Validate(); err != nil { return nil, err }
    return post, s.repo.Create(ctx, post)
}
```

### 4. Implement Adapters
```go
// internal/modules/{module}/adapters/sqlite_repository.go (OUTPUT ADAPTER)
type SQLiteRepository struct { db *sql.DB }
func (r *SQLiteRepository) Create(ctx context.Context, post *domain.Post) error {
    _, err := r.db.ExecContext(ctx, "INSERT INTO posts...", post.ID, post.Title)
    return err
}

// internal/modules/{module}/adapters/http_handler.go (INPUT ADAPTER)
type HTTPHandler struct { service ports.PostService }
func (h *HTTPHandler) CreatePostAPI(w http.ResponseWriter, r *http.Request) {
    // Parse request, call h.service.CreatePost(), write JSON response
}
```

### 5. Wire in `cmd/forum/wire/` (CRITICAL - Don't Skip!)
```go
// cmd/forum/wire/repositories.go
repos.Post = postAdapters.NewSQLitePostRepository(db)

// cmd/forum/wire/services.go (ServiceContainer already exists)
post: postApp.NewService(repos.Post, repos.Category),

// cmd/forum/wire/handlers.go
postHandler := postAdapters.NewHTTPHandler(services, templates)

// cmd/forum/wire/app.go (in initServer)
postHandler.RegisterRoutes(server.Router())
```

### 6. Database Migration (if schema changes)
```sql
-- migrations/NNN_{module}_{description}.sql
-- +migrate Up
CREATE TABLE posts (id TEXT PRIMARY KEY, title TEXT NOT NULL);
CREATE INDEX idx_posts_user_id ON posts(user_id);

-- +migrate Down
DROP INDEX idx_posts_user_id;
DROP TABLE posts;
```

**Migrations run automatically on startup** via `cmd/forum/wire/app.go:initDatabase()`.

---

## Unified Dependency Injection Pattern (CRITICAL)

**Every handler uses identical ServiceContainer pattern**:

```go
// cmd/forum/wire/services.go - ONE concrete container with ALL services
type ServiceContainer struct {
    auth     authPorts.AuthService      // Private fields
    user     userPorts.UserService
    post     postPorts.PostService
}
func (sc *ServiceContainer) Auth() authPorts.AuthService { return sc.auth }
func (sc *ServiceContainer) User() userPorts.UserService { return sc.user }

// Each handler constructor: IDENTICAL signature across ALL handlers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ertval) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
