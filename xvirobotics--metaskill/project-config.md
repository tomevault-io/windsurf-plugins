---
trigger: always_on
description: This is a fullstack web application built with React (frontend), Node.js/Express (backend), and PostgreSQL (database). The project uses TypeScript end-to-end, Prisma as the ORM, TanStack Query for server state management, and Tailwind CSS for styling. The architecture follows a monorepo structure with `client/` and `server/` directories sharing common types via a `shared/` package.
---

# CLAUDE.md

## Project Overview

This is a fullstack web application built with React (frontend), Node.js/Express (backend), and PostgreSQL (database). The project uses TypeScript end-to-end, Prisma as the ORM, TanStack Query for server state management, and Tailwind CSS for styling. The architecture follows a monorepo structure with `client/` and `server/` directories sharing common types via a `shared/` package.

## Agent Team

### Routing Table

| Task Type | Agent | When to Use |
|-----------|-------|-------------|
| Feature planning, task breakdown, architecture decisions | tech-lead | Any new feature, multi-step task, or when the best approach is unclear |
| React components, pages, hooks, styling, client-side logic | frontend-engineer | UI work, component creation, form handling, responsive design, accessibility |
| API endpoints, database schemas, migrations, auth, server logic | backend-engineer | REST APIs, Prisma models, JWT auth, validation, middleware, database queries |
| Docker, CI/CD, deployment, environment config, monitoring | devops-engineer | Dockerfiles, GitHub Actions, docker-compose, env management, health checks |
| Code quality, security review, performance audit | code-reviewer | All code changes before merge, PR reviews, refactor validation |

### Orchestration Protocol

1. **Tech-lead is the routing authority.** When a complex task arrives, the tech-lead agent analyzes it, breaks it into frontend/backend/devops subtasks, and delegates to the appropriate specialist(s).
2. **Main agent never implements directly** for multi-step tasks -- it delegates to specialists via the Task tool. Single-file trivial edits may be handled directly.
3. **Handoff format:** When delegating, provide: (a) clear objective, (b) relevant file paths, (c) acceptance criteria, (d) which agent to hand off to next.
4. **Max 2 agents in parallel** for complex tasks to avoid merge conflicts between frontend and backend changes.
5. **Code reviewer is the quality gate** -- all code changes pass through code-reviewer before the task is marked complete.

### Workflow Chains

- **New Feature**: tech-lead --> frontend-engineer + backend-engineer (parallel if independent) --> code-reviewer
- **Bug Fix**: tech-lead --> [relevant specialist] --> code-reviewer
- **Refactor**: tech-lead --> code-reviewer (review plan first) --> [specialist] --> code-reviewer (verify result)
- **Database Change**: tech-lead --> backend-engineer (migration + API) --> frontend-engineer (if UI affected) --> code-reviewer
- **DevOps Task**: tech-lead --> devops-engineer --> code-reviewer (review config)

## Coding Standards

### TypeScript
- Strict mode enabled (`strict: true` in tsconfig.json)
- No `any` types -- use `unknown` and narrow with type guards
- Prefer `interface` for object shapes, `type` for unions and intersections
- Use `as const` assertions for literal types
- Named exports only -- no default exports

### React (Frontend)
- Functional components only -- no class components
- Use `React.FC` sparingly; prefer explicit prop types
- Custom hooks prefixed with `use` and extracted to `hooks/` directory
- TanStack Query for all server state; local state with `useState`/`useReducer`
- Tailwind CSS for styling -- no inline style objects
- All interactive elements must be keyboard accessible
- Use `React.lazy` and `Suspense` for route-level code splitting

### Node.js (Backend)
- Express with typed request/response handlers
- Prisma ORM for all database access -- no raw SQL unless performance-critical
- Zod schemas for all request body/query validation
- JWT-based authentication with refresh token rotation
- Centralized error handling middleware with typed error classes
- All async route handlers wrapped in error-catching middleware

### Database
- Prisma migrations for all schema changes -- never edit the database directly
- Use UUIDs for primary keys
- All tables include `createdAt` and `updatedAt` timestamps
- Soft delete pattern with `deletedAt` nullable timestamp
- Index foreign keys and frequently queried columns

### File Structure
```
client/
  src/
    components/     # Reusable UI components
    pages/          # Route-level page components
    hooks/          # Custom React hooks
    lib/            # Utility functions
    types/          # Frontend-specific types
server/
  src/
    routes/         # Express route handlers
    middleware/     # Auth, validation, error handling
    services/       # Business logic layer
    prisma/         # Schema and migrations
    lib/            # Utility functions
    types/          # Backend-specific types
shared/
  types/            # Shared TypeScript interfaces and enums
```

## Workflow Discipline (All Agents)

### Planning
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately -- don't keep pushing
- Write detailed specs upfront to reduce ambiguity


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xvirobotics/metaskill](https://github.com/xvirobotics/metaskill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
