---
trigger: always_on
description: > **All AI agents must conform to [AI_POLICY.md](./AI_POLICY.md)**
---

# Arcane AI Agent Instructions

> **All AI agents must conform to [AI_POLICY.md](./AI_POLICY.md)**

Arcane is a modern Docker management UI with a **Go backend** (Echo router with Huma v2 typed API), **SvelteKit frontend** (Svelte 5), an optional headless agent, and a Cobra CLI. Three Go modules are unified via `go.work`: `backend/`, `cli/`, `types/`.

### Domain docs

Single-context domain docs live at `CONTEXT.md`;

## Development Environment

```bash
./scripts/development/dev.sh start    # Start Docker-based dev environment (hot reload)
./scripts/development/dev.sh stop|restart|rebuild|clean|logs
```

- Frontend: http://localhost:3000 (Vite HMR)
- Backend: http://localhost:3552 (Air hot reload)

## Architecture Overview

### Backend (`backend/`)

```
cmd/                  # backend entrypoint
api/                  # HTTP API surface
├── api.go            # Huma v2 setup mounted on Echo via humaecho
├── handlers/         # Huma handlers — thin wrappers that call services
├── middleware/       # Huma-specific auth bridge
└── ws/               # Echo WebSocket handlers
frontend/             # embedded SvelteKit build registration
internal/
├── bootstrap/        # App initialization, DI wiring, Echo router setup — START HERE
├── config/           # Environment configuration
├── database/         # GORM connection and migrations
├── middleware/       # Echo middleware: auth, CORS, env proxy, rate limiting
├── models/           # GORM database models (include BaseModel for UUID, timestamps)
└── services/         # Business logic — *_service.go files contain domain logic
pkg/
├── libarcane/        # Core reusable backend/domain libraries
├── projects/         # Compose project parsing and filesystem helpers
├── scheduler/        # Cron-backed background jobs
├── pagination/       # Search/filter/sort/pagination helpers
└── utils/            # Shared helper utilities
resources/            # migrations, images, fonts, email templates
```

**Key patterns:**

- Echo is the HTTP router. Do not add Gin code or Gin middleware.
- Huma v2 is still the typed REST/OpenAPI layer, mounted on Echo in `backend/api/api.go` with `humaecho.NewWithGroup`.
- Handlers are thin: extract typed input and auth context, call a service, return a typed response.
- Services receive dependencies via constructor injection (see [bootstrap.go](backend/internal/bootstrap/bootstrap.go))
- Router wiring and cross-cutting Echo middleware live in [router_bootstrap.go](backend/internal/bootstrap/router_bootstrap.go)
- Direct Echo routes are reserved for behavior that does not fit Huma cleanly, such as WebSockets, streaming/diagnostics, webhook trigger routes, Playwright-only routes, buildable auth routes, and the embedded frontend.
- Use `slog` for structured logging with context
- Error wrapping: `fmt.Errorf("context: %w", err)`

### API Layer (`backend/api/`)

```
api.go               # Huma config, schema naming, auth bridge, handler registration
handlers/*.go        # Resource handlers registered with huma.Register
middleware/auth.go   # Bridge from Huma middleware to auth services
ws/handler.go        # Echo WebSocket endpoints under /api/environments/:id/ws
diagnostics.go       # Direct Echo diagnostic routes
webhooks_trigger.go  # Public Echo webhook trigger route
```

Huma handlers use typed input/output structs with struct tags for validation:

```go
type ListContainersInput struct {
    EnvironmentID string `path:"id" doc:"Environment ID"`
    Search        string `query:"search" doc:"Search query"`
    Limit         int    `query:"limit" default:"20" doc:"Limit"`
}
```

Register new typed API handlers from `backend/api/handlers/` through `registerHandlers` in [api.go](backend/api/api.go). Use Echo groups only when the endpoint needs raw `echo.Context`, WebSockets, streaming, or custom middleware behavior that Huma cannot model.

### Frontend (`frontend/src/`)

```
routes/(app)/         # Main app pages (dashboard, containers, images, etc.)
routes/(auth)/        # Auth pages
lib/components/       # Reusable Svelte components (shadcn-svelte based)
lib/services/         # API service classes extending BaseAPIService
lib/stores/           # Svelte stores (*.store.svelte files use runes)
lib/types/            # TypeScript types
../messages/en.json   # Source i18n messages; Crowdin handles other locales
```

### Shared Types (`types/`)

Domain types shared between backend and CLI. Each domain has its own package (e.g., `types/container/`, `types/image/`).

### CLI (`cli/`)

The CLI is a Cobra application. Commands live under `cli/pkg/<domain>/`, shared command helpers live under `cli/internal/`, and public API shapes should come from `types/` instead of duplicating structs.

## Critical Patterns

### Svelte 5 ONLY — No Svelte 4 Syntax

```svelte
<!-- Props: use $props() -->
let { prop1, prop2 }: { prop1: string; prop2?: number } = $props();

<!-- State: use $state() -->
let count = $state(0);

<!-- Derived values: use $derived() or $derived.by() -->
let doubled = $derived(count * 2);
let computed = $derived.by(() => complexCalculation());

<!-- Side effects: use $effect() -->
$effect(() => { /* runs when dependencies change */ });
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getarcaneapp/arcane](https://github.com/getarcaneapp/arcane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
