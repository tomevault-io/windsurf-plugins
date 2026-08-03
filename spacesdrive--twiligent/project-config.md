---
trigger: always_on
description: Twiligent is a self-hosted social media analytics and publishing dashboard for creators and teams managing multiple YouTube channels and Instagram accounts. It is a private deployment product - the owner runs their own instance with their own API credentials. There is no SaaS model, no multi-tenancy abstraction, and no billing system.
---

# Twiligent - AI Operating Manual

## Project Identity

Twiligent is a self-hosted social media analytics and publishing dashboard for creators and teams managing multiple YouTube channels and Instagram accounts. It is a private deployment product - the owner runs their own instance with their own API credentials. There is no SaaS model, no multi-tenancy abstraction, and no billing system.

- Primary users: Small creator teams, agencies, individual power users
- Primary value: Unified analytics and scheduled Instagram publishing across multiple accounts
- Current platforms: YouTube (analytics only), Instagram (analytics and publishing)
- GitHub repository: https://github.com/spacesdrive/twiligent

---

## Reading Order Before Any Change

Before implementing any feature or fix, read these documents in this exact order:

1. This file - project identity, rules, and documentation map
2. `docs/WRITING_STANDARDS.md` - typography, icons, writing style, UI copy rules
3. `docs/architecture/OVERVIEW.md` - system topology and request flow
4. `docs/guidelines/JAVASCRIPT.md` - code standards (always apply)
5. Relevant architecture doc - backend, frontend, database, or cloudflare
6. Relevant feature guide - if one exists in `docs/features/`
7. Relevant engineering guideline - if the task touches a specific layer

For MCP usage, read `docs/mcp/OVERVIEW.md` before using any MCP server.

---

## Documentation Map

```
docs/
  WRITING_STANDARDS.md     Typography, icons, writing style, UI copy, commit messages
  architecture/
    OVERVIEW.md            System topology, connections, request lifecycle
    backend/
      HONO.md              App structure, middleware, routing pattern
      ROUTES.md            All routes, method, auth requirement, purpose
      SERVICES.md          instagram.js, youtube.js service layer
      MIDDLEWARE.md        requireAuth, CORS, context injection
      CACHING.md           Redis strategy, cache keys, invalidation
      CRON.md              Scheduled handlers, cron triggers
    frontend/
      REACT_ARCHITECTURE.md    Provider tree, lazy loading, App.jsx
      ROUTING.md               All pages, paths, layout nesting
      CONTEXTS.md              AuthContext, AppContext, usage rules
      API_LAYER.md             api.js, request(), auth header injection
      FEATURES.md              Feature module conventions, directory map
    database/
      SCHEMA.md            All tables, columns, types, constraints
      SECURITY.md          RLS policy, service vs anon key, isolation
    cloudflare/
      WORKERS.md           Worker export, env binding, cron handler
      PAGES.md             Frontend deployment, build step, env vars
      CI_CD.md             GitHub Actions workflows, path filters
    data-flows/
      AUTHENTICATION.md    Login, JWT lifecycle, session management
      PUBLISHING.md        Cloudinary to IG container to publish pipeline
      ANALYTICS.md         YouTube and Instagram analytics fetch and compute
      SCHEDULING.md        Scheduled post lifecycle, dual-scheduler design
  guidelines/
    JAVASCRIPT.md          Code style, modules, naming, comments policy
    REACT.md               Component rules, hooks, JSX conventions
    HONO.md                Route handlers, middleware, context usage
    CLOUDFLARE_WORKERS.md  Worker-safe patterns, env access, no Node APIs
    SUPABASE.md            Client usage, query patterns, service vs anon
    SHADCN.md              Component usage, composition rules, imports
    TAILWIND.md            Utility usage, token references, responsive
    NAMING.md              File names, function names, variable names
    ERROR_HANDLING.md      Error boundaries, API errors, logging
    STATE_MANAGEMENT.md    Context vs local state, when to lift
    API_CONVENTIONS.md     Response shapes, error shapes, status codes
  mcp/
    OVERVIEW.md            When to use each MCP, decision tree
    CONTEXT7.md            Library docs, API reference lookups
    FILESYSTEM.md          File search, symbol location, refactoring
    SEQUENTIAL_THINKING.md Complex planning, architecture decisions
    PARALLEL_SEARCH.md     UX research, best practices, browser compat
  workflows/
    FEATURE_DEVELOPMENT.md 25-step feature implementation sequence
    TESTING.md             Test types, commands, pass criteria
    GIT.md                 Commit format, branch rules, changelog
    DEPLOYMENT.md          Backend deploy, frontend deploy, verification
  features/
    NEW_ANALYTICS_PAGE.md  Add analytics view for a new platform
    NEW_API_ENDPOINT.md    Add a route to the Hono backend
    NEW_REACT_PAGE.md      Add a page to the React SPA
    NEW_SCHEDULED_TASK.md  Add a new cron handler to the Worker
    NEW_DATABASE_TABLE.md  Add a Supabase table with migrations
    NEW_INTEGRATION.md     Add a third-party platform (API + OAuth)
  philosophy/
    ARCHITECTURE.md        Architectural principles and constraints
    SECURITY.md            Security model, key handling, isolation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spacesdrive/twiligent](https://github.com/spacesdrive/twiligent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
