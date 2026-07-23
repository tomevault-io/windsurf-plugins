---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Comment Style

**Always use ultra-brief mode for all PR reviews and responses.**

Format:

- Critical issues only (bugs, security, blockers)
- Brief bullet points, no lengthy explanations
- Skip verbose sections (no "Strengths", "Summary", etc.)
- Include file:line references when relevant
- Maximum ~10-15 lines per response

## Testing Unreleased PRs

When replying to a GitHub issue or discussion where the fix lives in an open PR, ask the reporter to test the pre-built image: `amir20/dozzle:pr-XXX` (XXX = PR number). CI builds a tagged image per PR, so reporters can verify without waiting for the next release.

## GitHub Tone (issues, PRs, comments, discussions)

When posting anything to GitHub, write like a human maintainer, not an AI assistant. Avoid telltale LLM patterns:

- No em dashes or en dashes. Use commas, periods, or parentheses instead.
- No "Not X, but Y" rhetorical contrasts.
- No throat-clearing openers ("Great point", "Makes sense", "Thanks for the detailed write-up").
- No closing summaries or recap sentences.
- No bolded inline labels mid-paragraph ("**Why:**", "**Note:**").
- Drop hedges ("essentially", "basically", "essentially just"). Say it plain.
- Lowercase casual tone is fine. Contractions are fine. Short sentences are fine.
- Don't over-explain tradeoffs. State the decision, give one reason, stop.

## Project Overview

Dozzle is a lightweight, web-based Docker log viewer with real-time monitoring capabilities. It's a hybrid application with:

- **Backend**: Go (HTTP server, Docker API client, WebSocket streaming)
- **Frontend**: Vue 3 (SPA with Vite, TypeScript)

The application supports multiple deployment modes: standalone server, Docker Swarm, and Kubernetes (k8s).

## Development Commands

### Setup

```bash
# Install dependencies
pnpm install

# Generate certificates and protobuf files
make generate
```

### Development

```bash
# Run full development environment (backend + frontend with hot reload)
make dev

# Alternative: Run backend and frontend separately
pnpm run watch:backend  # Go backend with air (port 3100)
pnpm run watch:frontend # Vite dev server (port 3100)

# Run in agent mode for development
pnpm run agent:dev
```

### Building

```bash
# Build frontend assets
pnpm build
# or
make dist

# Build entire application (includes frontend build)
make build

# Build Docker image
make docker
```

### Testing

```bash
# Run Go tests
make test

# Run frontend tests (Vitest)
pnpm test
# Run in watch mode
TZ=UTC pnpm test --watch

# Type checking
pnpm typecheck
```

### Preview & Other

```bash
# Preview production build locally
pnpm preview
# or
make preview

# Run integration tests (Playwright)
make int
```

## Architecture

### Backend (Go)

The Go backend is organized into these key packages:

- **`internal/web/`** - HTTP server and routing layer
  - Routes defined in `routes.go` using chi router
  - WebSocket/SSE handlers for log streaming (`logs.go`)
  - Authentication middleware and token management (`auth.go`)
  - Container action handlers (`actions.go`)

- **`internal/docker/`** - Docker API client implementation
  - `client.go`: Main Docker client wrapper with container operations
  - `log_reader.go`: Streaming container logs
  - `stats_collector.go`: Real-time container stats collection

- **`internal/agent/`** - gRPC agent for multi-host support
  - Uses Protocol Buffers (protos defined in `protos/`)
  - Enables distributed log collection across Docker hosts

- **`internal/cloud/`** - Dozzle Cloud integration (tool execution engine)
  - `client.go`: Bidirectional gRPC stream client with auto-reconnect and exponential backoff
  - `tools.go`: Tool registration, dispatch (`executeTool`), and `ToolHostService` interface
  - `tools_containers.go`: Container listing, finding, stats, and inspection tools
  - `tools_logs.go`: Log fetching with level/query/regex filtering (max 100 lines)
  - `tools_actions.go`: Container start/stop/restart actions (gated by `enableActions`)
  - `tools_helpers.go`: Proto conversion utilities and host name resolution
  - Uses `protos/cloud.proto` for service and message definitions

- **`internal/k8s/`** - Kubernetes client support
  - Alternative to Docker client for k8s deployments

- **`internal/support/`** - Support utilities
  - `cli/`: Command-line argument parsing and validation
  - `docker/`: Multi-host Docker management and Swarm support (`docker_service.go`, client managers)
  - `k8s/`: Kubernetes service abstractions
  - `web/`: Web service utilities

- **`internal/auth/`** - Authentication providers
  - Simple file-based auth (`simple.go`)
  - Forward proxy auth (`proxy.go`)
  - Role-based authorization (`roles.go`)

- **`internal/container/`** - Container domain models and interfaces
  - `event_generator.go`: Log parsing and grouping logic (multi-line, JSON detection)

- **`internal/notification/`** - Alert and notification system
  - `manager.go`: Notification rule evaluation and dispatching
  - `log_listener.go`: Log pattern matching for alerts
  - `dispatcher/`: Notification channel implementations (email, webhook, etc.)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amir20/dozzle](https://github.com/amir20/dozzle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
