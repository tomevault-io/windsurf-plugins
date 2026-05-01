---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Team9 is a full-stack instant messaging and team collaboration platform built as a monorepo. The backend uses NestJS with PostgreSQL (Drizzle ORM), while the frontend is a Tauri-based cross-platform desktop app (React + TypeScript) with real-time WebSocket communication via Socket.io.

## Common Commands

### Development

```bash
pnpm dev              # Run server (gateway + im-worker) and client concurrently
pnpm dev:client       # Web frontend only (Vite dev server)
pnpm dev:desktop      # Tauri desktop app (hot reload)
pnpm dev:server       # Gateway service only
pnpm dev:im-worker    # Background IM worker service only
pnpm dev:server:all   # Both gateway and im-worker services
```

> Note: `pnpm dev` and other scripts use Turborepo for task orchestration.
> Build artifacts are cached locally in `.turbo/`. Run `turbo run build`
> directly if you need fine-grained control (e.g., `--filter`, `--dry`).

### Database Operations

```bash
pnpm db:generate      # Generate Drizzle schemas from TypeScript
pnpm db:migrate       # Run pending migrations
pnpm db:push          # Push schema changes to database (dev only)
pnpm db:studio        # Open Drizzle Studio UI for database inspection
```

### Building

```bash
pnpm build                 # Build both server and client
pnpm build:server          # Build NestJS backend
pnpm build:client          # Build web client
pnpm build:client:mac      # Build macOS Tauri app
pnpm build:client:windows  # Build Windows Tauri app
```

### Production

```bash
pnpm start:prod       # Start server in production mode
```

## Architecture

### Monorepo Structure

```
apps/
├── client/           # Tauri + React frontend
├── server/
│   ├── apps/
│   │   ├── gateway/  # Main API gateway (port 3000)
│   │   └── im-worker/  # Background IM worker service (port 3001)
│   └── libs/         # Shared libraries
│       ├── database/ # Drizzle schemas and DB module
│       ├── auth/     # Shared authentication
│       ├── ai-client/
│       ├── agent-framework/
│       ├── redis/
│       ├── rabbitmq/
│       └── shared/   # Common types, constants
└── debugger/         # Debug tool
```

### Backend Architecture (NestJS)

**Entry Point:** [apps/server/apps/gateway/src/main.ts](apps/server/apps/gateway/src/main.ts)

The backend follows a modular NestJS architecture with two main applications:

- **Gateway:** Main API service with REST endpoints and WebSocket gateway
- **IM Worker:** Background service for async processing (message persistence, routing, offline delivery)

**Key Modules:**

- **Auth Module** ([apps/server/apps/gateway/src/auth](apps/server/apps/gateway/src/auth)): JWT-based authentication with Passport strategy, 7-day token expiry
- **IM Module** ([apps/server/apps/gateway/src/im](apps/server/apps/gateway/src/im)): Instant messaging functionality
  - Channels: direct, public, private types
  - Messages: text, file, image, system, long_text types with threading support (parentId)
  - Long text: messages >=20 lines or >=2000 chars auto-classified as `long_text`, truncated at API layer, full content via `GET /messages/:id/full-content`
  - Users: profile management, status tracking
  - Properties: channel property definitions, message property values (16 types), AI auto-fill
  - Views: table/board/calendar views, channel tabs
  - Audit: change audit logging for channels, messages, and properties
  - WebSocket: Socket.io gateway for real-time events
- **Workspace Module** ([apps/server/apps/gateway/src/workspace](apps/server/apps/gateway/src/workspace)): Multi-tenant workspace management
- **Edition Module** ([apps/server/apps/gateway/src/edition](apps/server/apps/gateway/src/edition)): Dynamic feature loading for Community vs Enterprise editions

**Edition System:**
The codebase supports Community and Enterprise editions via environment variable `EDITION=community|enterprise`. The Edition module conditionally loads features (e.g., TenantModule only in enterprise). Enterprise code lives in a separate git submodule at `enterprise/`.

**Database Layer:**

- Uses Drizzle ORM with PostgreSQL
- Schemas organized by domain in [apps/server/libs/database/schemas](apps/server/libs/database/schemas):
  - **im/**: users, channels, messages, channel_members, message_attachments, message_reactions, message_acks, mentions, user_channel_read_status, channel_property_definitions, message_properties, audit_logs, channel_views, channel_tabs
  - **tenant/**: tenants, tenant_members, workspace_invitations
- All migrations managed via `pnpm db:migrate`
- Schema changes pushed via `pnpm db:push` (dev) or `pnpm db:generate` + `pnpm db:migrate` (prod)

### Frontend Architecture (Tauri + React)

**Entry Point:** [apps/client/src/main.tsx](apps/client/src/main.tsx)

**State Management:**

- **Zustand** for UI state: theme, user profile, loading states
  - App store: [apps/client/src/stores/app.ts](apps/client/src/stores/app.ts)
  - Workspace store: [apps/client/src/stores/workspace.ts](apps/client/src/stores/workspace.ts)
  - Home store: [apps/client/src/stores/home.ts](apps/client/src/stores/home.ts)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [team9ai/team9](https://github.com/team9ai/team9) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
