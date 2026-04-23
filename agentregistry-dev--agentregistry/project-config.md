---
trigger: always_on
description: This document provides guidelines for both AI coding assistants (Claude, Copilot, Cursor, etc.) and human developers working on the AgentRegistry codebase.
---

# AGENTS.md - Development Guidelines

This document provides guidelines for both AI coding assistants (Claude, Copilot, Cursor, etc.) and human developers working on the AgentRegistry codebase.

## Project Overview

AgentRegistry is a centralized registry for securely curating, discovering, deploying, and managing agentic infrastructure including MCP (Model Context Protocol) servers, agents, and skills.

**Tech Stack:**
- **Backend/CLI:** Go 1.25+
- **Database:** PostgreSQL with pgvector (accessed via pgx)
- **Frontend:** Next.js 14 (App Router) with Tailwind CSS
- **CLI Framework:** Cobra
- **API Framework:** Huma (OpenAPI)

---

## Architecture

### Directory Structure

```
agentregistry/
├── cmd/           # Entry points only - minimal code
│   ├── cli/       # CLI entry point
│   └── server/    # Server entry point
├── pkg/           # Public, reusable packages
├── internal/      # Private implementation
│   ├── registry/  # Core registry implementation
│   │   ├── api/       # HTTP handlers
│   │   ├── database/  # Database layer (pgx)
│   │   ├── service/   # Business logic
│   │   └── ...
│   ├── cli/       # CLI command implementations
│   ├── mcp/       # MCP protocol handling
│   └── daemon/    # Daemon orchestration
├── ui/            # Next.js frontend
└── docker/        # Container configurations
```

### Layer Responsibilities

1. **cmd/** - Entry points only. Delegate immediately to pkg/ or internal/
2. **pkg/** - Public APIs for external consumption and reusability
3. **internal/** - Implementation details, not importable by external packages
4. **internal/registry/database/** - **ONLY** place that accesses the database directly
5. **internal/registry/service/** - Business logic, receives database interface via constructor

### Platform Ownership

Deployment/platform code should be organized by **clear ownership**, not by vague helper layers:

1. **`internal/registry/platforms/<platform>/` owns platform behavior** - local and kubernetes packages should contain their adapter plus the concrete platform-specific materialization/apply/discovery logic they need
2. **`internal/registry/platforms/utils/` is for narrowly shared deployment utilities only** - use it for adapter-shared materialization helpers, validation, name generation, and request parsing that are truly cross-platform
3. **`internal/registry/platforms/types/` is for shared contracts only** - keep shared schemas and DTO-style platform types here, not behavior-heavy logic
4. **`internal/registry/api/handlers/` is transport only** - HTTP handlers should parse requests, call services/adapters, and map errors; they should not own deployment/platform behavior
5. **`internal/registry/registry_app.go` is the composition root** - wire concrete platform adapters here explicitly instead of hiding registration/factory behavior in handler packages

Avoid introducing broad "translator" layers or catch-all shared packages when the code really belongs to one concrete platform. Prefer concentrated platform packages with small, explicit shared utilities.

---

## Critical Rules

### Database Access

**The database MUST only be accessed by:**
1. The database layer (`internal/registry/database/`)
2. The authorizer component

**No other component should have direct database access.** All data operations must go through the appropriate interfaces.

```go
// CORRECT: Service receives database interface
type RegistryService struct {
    db DatabaseInterface
}

func NewRegistryService(db DatabaseInterface) *RegistryService {
    return &RegistryService{db: db}
}

// INCORRECT: Service creates its own database connection
type RegistryService struct {
    db *pgxpool.Pool  // Direct database access - DO NOT DO THIS
}
```

### Interface Design

Every significant component must define an interface for its dependencies. This enables:
- Unit testing with mocks
- Loose coupling between packages
- Clear contract definitions

```go
// Define interfaces for dependencies
type AgentRepository interface {
    GetAgent(ctx context.Context, id string) (*Agent, error)
    ListAgents(ctx context.Context, opts ListOptions) ([]Agent, error)
    CreateAgent(ctx context.Context, agent *Agent) error
}

// Implementation receives interface, not concrete type
type AgentService struct {
    repo AgentRepository
}
```

### Single Responsibility

Each package and file should have one clear purpose. Signs of mixed responsibilities:
- Files over 500 lines
- Packages importing many unrelated dependencies
- Functions doing multiple unrelated things

**Split large components into focused units.**

---

## Error Handling

Use standard Go error patterns with wrapping:

```go
import (
    "errors"
    "fmt"
)

// Wrap errors with context
func (s *Service) GetAgent(ctx context.Context, id string) (*Agent, error) {
    agent, err := s.repo.GetAgent(ctx, id)
    if err != nil {
        return nil, fmt.Errorf("getting agent %s: %w", id, err)
    }
    return agent, nil
}

// Check error types
if errors.Is(err, ErrNotFound) {
    // handle not found
}

// Define sentinel errors for domain-specific cases
var (
    ErrNotFound     = errors.New("not found")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentregistry-dev/agentregistry](https://github.com/agentregistry-dev/agentregistry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
