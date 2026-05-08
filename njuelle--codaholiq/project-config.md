---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Codaholiq — an AI automations governance platform for GitHub repositories. Users configure triggers (GitHub events, cron, manual), choose an AI provider (Claude Code, OpenAI Codex, Gemini CLI, or OpenCode), write prompt templates, and the platform dispatches GitHub Actions workflows to the selected provider. Executions are tracked with real-time log streaming and cost analytics.

## Architecture

```
apps/
  api/    # NestJS 11 backend (HTTP + BullMQ processors + Drizzle ORM)
  web/    # React 19 + Vite + Tailwind v4 + shadcn/ui frontend
```

Single NestJS process handles both REST API and BullMQ job processing via `@nestjs/bullmq`. No separate worker. No shared packages — the REST API is the contract between backend and frontend.

### Backend layout (`apps/api/src/`)

```
database/
  database.module.ts          # Drizzle client provider
  drizzle.config.ts           # drizzle-kit config (reads schema.ts barrel)
  migrations/                 # Generated migrations
  schema.ts                   # Barrel re-exporting ALL *.schema.ts from modules
  test/                       # DB test utilities & seed factories
modules/
  auth/                       # GitHub OAuth, JWT, refresh tokens
  organizations/              # Org CRUD, members, dashboard
  webhooks/                   # GitHub webhook receiver + signature guard
  github/                     # GitHub API client, repo sync, installation mgmt
  automations/                # Automation CRUD
    triggers/                 # Event matching, cron scheduling, conditions
    templates/                # Prompt template engine, safety utils
    dto/                      # Automation DTOs
  executions/                 # Execution tracking, BullMQ processors, SSE logs, cost extraction
  providers/                  # AI provider registry, model catalog, dispatch input mapping
  permissions/                # Role-based permissions
  audit/                      # Audit logging
  notifications/              # User notifications
  variables/                  # Custom shared variables
  health/                     # Health check endpoint
common/
  guards/                     # jwt-auth, org, permission, throttler
  pipes/                      # zod-validation
  filters/                    # http-exception
  interceptors/               # transform, logging, audit
  decorators/                 # @CurrentUser, @Public, @RequirePermission, @Audit
  crypto/                     # Env validation, secret masking
  sanitization/               # Input sanitization
  logging/                    # Logging module
  monitoring/                 # Job failure tracking
```

Each module owns its Drizzle schema, repository, service, controller, and DTOs. Schemas are co-located in their module (not in a central folder). Large modules may have sub-directories grouping related services (e.g., `automations/triggers/`, `automations/templates/`).

### Frontend layout (`apps/web/src/`)

Domain-driven structure — each domain module owns its pages, components, hooks, types, and lib.

```
modules/                        # Domain modules
  auth/                         # Login, OAuth callback, org selector
  automations/                  # Automation CRUD, forms, trigger config
  executions/                   # Execution list, detail, log viewer
  dashboard/                    # Dashboard stats, charts
  organizations/                # Org settings, members management
  repositories/                 # Repository list, sync
  permissions/                  # Permission management UI
  notifications/                # Notification bell, hooks
  variables/                    # Shared variables management
common/                         # Shared, cross-domain code
  components/
    ui/                         # shadcn/ui primitives (button, input, card, ...)
    layout/                     # App shell, sidebar, top bar
    *.tsx                       # Shared components (page-header, status-badge, ...)
  hooks/                        # Utility hooks (use-clipboard, use-debounce, ...)
  lib/                          # API client, query keys, utils, format
  pages/                        # Error pages (not-found, forbidden, server-error)
  types/                        # Shared API types + barrel re-exporting all module types
test/                           # Test utilities (setup, MSW handlers, factories)
```

**Each domain module follows this internal structure:**
```
modules/<domain>/
  pages/                        # Route-level components
    __tests__/                  # Page tests
  components/                   # Domain-specific UI components
    __tests__/                  # Component tests
    form/                       # Sub-group if needed (e.g., multi-step forms)
      __tests__/
  hooks/                        # Domain hooks (data fetching, mutations)
    __tests__/                  # Hook tests
  lib/                          # Domain-specific utilities (optional)
    __tests__/
  types.ts                      # Domain type definitions
```

## File naming & test conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Njuelle/Codaholiq](https://github.com/Njuelle/Codaholiq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
