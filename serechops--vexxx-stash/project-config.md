---
trigger: always_on
description: > **Model Target**: Claude 4.x (Sonnet 4.5, Opus 4.5, Haiku 4.5)
---

# Vexxx (Stash Fork) - AI Coding Instructions

> **Model Target**: Claude 4.x (Sonnet 4.5, Opus 4.5, Haiku 4.5)
>
> This document provides comprehensive context for Claude-based AI assistants working on the Vexxx codebase. It follows Claude 4 prompting best practices for precise instruction following and effective agentic coding.

---

## Project Overview

Vexxx is a **fork of Stash**, a self-hosted web application written in **Go** (backend) and **React/TypeScript** (frontend). It organizes and serves media collections with advanced metadata management, scraping capabilities, and extensible plugin architecture.

### Core Technologies

| Layer | Technology | Notes |
|-------|------------|-------|
| Backend | Go 1.24+ | Chi router, SQLite, GraphQL (gqlgen) |
| Frontend | React 18 + TypeScript | Material UI, Apollo Client, Vite |
| Database | SQLite | Embedded, migrations via golang-migrate |
| API | GraphQL | Schema-first with gqlgen code generation |
| Media Processing | FFmpeg | Transcoding, thumbnails, sprites |

---

## Architecture Overview

<architecture_context>
Vexxx follows a clean separation between API layer, business logic, and data access:

```
├── cmd/stash/           # Application entrypoint
├── graphql/schema/      # GraphQL schema definitions (source of truth)
├── internal/
│   ├── api/             # GraphQL resolvers, HTTP routes, server
│   ├── manager/         # Core business logic, system status
│   ├── autotag/         # Auto-tagging engine
│   └── identify/        # Content identification
├── pkg/
│   ├── models/          # Domain models and interfaces
│   ├── sqlite/          # SQLite repository implementations
│   ├── scraper/         # Scraping framework
│   ├── plugin/          # Plugin system
│   └── ffmpeg/          # FFmpeg wrapper
└── ui/v2.5/
    ├── graphql/         # Frontend GraphQL operations
    └── src/
        ├── components/  # React components
        ├── core/        # Generated GraphQL hooks
        └── hooks/       # Custom React hooks
```
</architecture_context>

---

## GraphQL Development Workflow

<graphql_workflow>
Vexxx uses a **Schema-First** approach. The GraphQL schema is the source of truth.

### Implementation Cycle

1. **Define Schema**: Add types/queries/mutations to `graphql/schema/schema.graphql` or type-specific files in `graphql/schema/types/`
2. **Generate Go Code**: Run `make generate` to update `internal/api/generated_models.go`
3. **Implement Resolvers**: Create resolver in `internal/api/resolver_*.go`
4. **Rebuild Backend**: Run `make stash` - schema changes require binary rebuild
5. **Define Frontend Operations**: Create `.graphql` file in `ui/v2.5/graphql/queries/`
6. **Generate Frontend Hooks**: Run `pnpm run generate` in `ui/v2.5/`
7. **Use in Components**: Import from `src/core/generated-graphql`

### Schema Locations

- Main schema: `graphql/schema/schema.graphql`
- Type definitions: `graphql/schema/types/*.graphql`
- Frontend queries: `ui/v2.5/graphql/queries/**/*.graphql`
- Frontend mutations: `ui/v2.5/graphql/mutations/**/*.graphql`
</graphql_workflow>

---

## Database Patterns

<database_patterns>
### SQLite with golang-migrate

- Schema version tracked in `pkg/sqlite/database.go` (`appSchemaVersion`)
- Migrations embedded via `//go:embed migrations/*.sql`
- Repository pattern: interfaces in `pkg/models/`, implementations in `pkg/sqlite/`

### Key Tables

| Entity | Model File | Repository |
|--------|-----------|------------|
| Scene | `pkg/models/model_scene.go` | `pkg/sqlite/scene.go` |
| Performer | `pkg/models/model_performer.go` | `pkg/sqlite/performer.go` |
| Studio | `pkg/models/model_studio.go` | `pkg/sqlite/studio.go` |
| Tag | `pkg/models/model_tag.go` | `pkg/sqlite/tag.go` |
| Gallery | `pkg/models/model_gallery.go` | `pkg/sqlite/gallery.go` |

### Transaction Pattern

```go
// Use txn.WithDatabase for transactional operations
err := txn.WithDatabase(ctx, r.repository, func(ctx context.Context) error {
    // All operations here are transactional
    return nil
})
```
</database_patterns>

---

## Frontend Architecture

<frontend_architecture>
### Component Organization

```
ui/v2.5/src/components/
├── Scenes/           # Scene list, detail, edit views
├── Performers/       # Performer management
├── Settings/         # Configuration UI
├── ScenePlayer/      # Video player with segment support
├── Tagger/           # Metadata identification
├── Shared/           # Reusable components
└── FrontPage/        # Landing page with carousels
```

### State Management

- **Apollo Client**: Primary data fetching via generated hooks
- **React Context**: Auth state, theme, read-only mode
- **Local State**: Component-specific with useState/useReducer

### Generated Hooks Pattern

```typescript
// Use generated hooks from core/generated-graphql
import * as GQL from "src/core/generated-graphql";

// Queries
const { data, loading } = GQL.useFindScenesQuery({ variables: { filter } });

// Mutations
const [updateScene] = GQL.useSceneUpdateMutation();

// Lazy queries for on-demand fetching
const [validatePath] = GQL.useValidateLibraryPathLazyQuery();
```

### Internationalization

- Locale files in `ui/v2.5/src/locales/`
- Primary: `en-GB.json`
- Use `<FormattedMessage id="key.path" />` or `intl.formatMessage()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Serechops/vexxx-stash](https://github.com/Serechops/vexxx-stash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
