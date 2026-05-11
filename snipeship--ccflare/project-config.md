---
trigger: always_on
description: Guidance for code agents working in this repository.
---

# AGENTS.md

Guidance for code agents working in this repository.

This file is the repo-local source of truth for:

- workspace structure
- package ownership
- commands to run before handing work back
- DRY / refactor expectations
- where to look in `docs/` before making architectural changes

## Engineering Standard

Always optimize for code that is:

- clear to read
- easy to maintain
- appropriately DRY
- locally coherent
- boring in the good sense

Prefer small, obvious abstractions over clever ones. Remove repetition when it
reduces maintenance cost, but do not introduce generic helper layers that make
the code harder to follow than the duplication they replace.

The target is not “maximum abstraction.” The target is code that a strong
engineer can understand and modify quickly without guessing at hidden behavior.

## What This Repo Is

`ccflare` is a Bun/TypeScript monorepo for a multi-provider AI proxy.

Core product behavior:

- provider-native HTTP passthrough for `anthropic`, `openai`, `claude-code`, and `codex`
- WebSocket proxying for supported upstreams
- account failover and session-based selection
- SQLite-backed request/account/auth-session persistence
- management API
- browser dashboard
- terminal UI

High-level docs:

- [`README.md`](README.md)
- [`docs/index.md`](docs/index.md)
- [`docs/architecture.md`](docs/architecture.md)
- [`docs/data-flow.md`](docs/data-flow.md)

## Reading Order

If you are changing behavior, read in this order:

1. [`README.md`](README.md)
2. [`docs/architecture.md`](docs/architecture.md)
3. the domain-specific doc for your area

Use these targeted references:

- API/control plane: [`docs/api-http.md`](docs/api-http.md)
- runtime/request flow: [`docs/data-flow.md`](docs/data-flow.md)
- provider behavior: [`docs/providers.md`](docs/providers.md)
- config rules: [`docs/configuration.md`](docs/configuration.md)
- database/persistence: [`docs/database.md`](docs/database.md)
- load-balancing/session strategy: [`docs/load-balancing.md`](docs/load-balancing.md)
- TUI behavior: [`docs/tui.md`](docs/tui.md)
- deployment/paths/build outputs: [`docs/deployment.md`](docs/deployment.md)
- contribution expectations: [`docs/contributing.md`](docs/contributing.md)

## Current Workspace Layout

```text
ccflare/
├── apps/
│   ├── desktop/         # Desktop shell
│   ├── lander/          # Landing page
│   ├── server/          # Main Bun server entrypoint
│   ├── tui/             # Terminal UI + local TUI core
│   └── web/             # Browser dashboard
├── packages/
│   ├── api/             # REST/SSE handler layer and API transport types
│   ├── config/          # Config loading/defaults/env/path handling
│   ├── core/            # Constants, validation, lifecycle, DI, pricing
│   ├── database/        # SQLite schema, repositories, async writer
│   ├── http/            # Shared HTTP client/response/header/error utilities
│   ├── logger/          # Logging, log bus, file writer
│   ├── oauth-flow/      # Shared OAuth onboarding flow
│   ├── providers/       # Provider registry + provider implementations
│   ├── proxy/           # HTTP/WebSocket proxy path
│   ├── runtime-server/  # Server bootstrap/orchestration
│   ├── types/           # Shared domain and transport types
│   └── ui/              # Shared UI presenters/components/constants
└── docs/
```

## Package Ownership

Use these ownership rules when deciding where code belongs.

### `apps/server`

Owns:

- the top-level server entrypoint
- thin wrapper around `@ccflare/runtime-server`

Do not move orchestration logic here if it can live in `runtime-server`.

### `apps/tui`

Owns:

- the TUI app
- TUI-only core wrappers under `apps/tui/src/core`
- screen components and terminal interaction behavior

TUI-local logic should stay here unless it is reused by another app.

### `apps/web`

Owns:

- the browser dashboard
- dashboard-only chart composition and page models
- static web build output

Shared UI display logic belongs in `@ccflare/ui`, not here.

### `packages/runtime-server`

Owns:

- runtime bootstrap
- Bun server startup
- request routing between `api`, `proxy`, and `web`
- startup maintenance
- graceful shutdown

This package is intentionally orchestration-heavy. Do not turn it into a grab-bag for provider or API logic.

### `packages/api`

Owns:

- management/control-plane handlers
- `APIRouter`
- API transport response types specific to the HTTP API surface

Management endpoint behavior belongs here.

### `packages/proxy`

Owns:

- provider-path request forwarding
- websocket proxy handling
- retry/failover behavior
- session strategy implementation
- request event emission
- worker handoff for usage/payload processing

Data-plane request forwarding belongs here.

### `packages/providers`

Owns:

- provider-specific URL construction
- auth header preparation
- OAuth provider adapters
- token refresh behavior
- provider-native rate-limit parsing
- provider-specific usage parsing

Avoid moving app/session orchestration into provider packages.

### `packages/oauth-flow`

Owns:

- shared OAuth onboarding flow for both API and TUI
- auth session state orchestration
- PKCE setup and completion flow

Keep it separate while both `@ccflare/api` and `ccflare` use it.

### `packages/database`

Owns:

- schema
- migrations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [snipeship/ccflare](https://github.com/snipeship/ccflare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
