---
trigger: always_on
description: > This file is human-curated project knowledge for AI agents.
---

# AGENTS.md — mayar-cli

> This file is human-curated project knowledge for AI agents.
> Agents may propose updates, but humans approve them.
> Research shows human-written AGENTS.md improves agent success ~4%.

---

## Project Overview

- **Name**: mayar-cli
- **Type**: api
- **Stack**: node-typescript
- **Description**: TODO - describe what this project does

---

## Conventions

### File Structure
```
TODO - document your project's file structure pattern
Example:
src/
├── routes/       # API route handlers
├── services/     # Business logic
├── models/       # Database models/schemas
├── middleware/    # Express/framework middleware
├── utils/        # Shared utilities
└── types/        # TypeScript type definitions
```

### Naming Conventions
- Files: `kebab-case.ts`
- Components: `PascalCase.tsx`
- Functions: `camelCase`
- Constants: `UPPER_SNAKE_CASE`
- Database tables: `snake_case`
- API endpoints: `kebab-case`

### Code Patterns
- TODO - document patterns used in this project
- Example: "All route handlers follow: validate input -> call service -> format response"
- Example: "Use Zod for all runtime validation"
- Example: "Error responses always use format: { error: string, code: string }"

### Testing Conventions
- Framework: none
- Command: echo 'no test command configured'
- Pattern: TODO - describe your test patterns
- Example: "Integration tests use a real test database, not mocks"
- Example: "Each test file has its own setup/teardown"

---

## Known Gotchas

- TODO - document things that are surprising or non-obvious
- Example: "Prisma unique constraint errors throw P2002, need explicit handling"
- Example: "Test database needs cleanup between runs — use beforeEach, not beforeAll"
- Example: "The auth middleware reads from both cookie and Authorization header"

---

## Architecture Decisions

- TODO - document why things are the way they are
- Example: "We use JWT instead of sessions because the API serves mobile + web"
- Example: "Chose Drizzle over Prisma for better type inference and SQL control"
- Example: "Monorepo with turborepo because shared types between API and web"

---

## Dependencies & Integrations

- TODO - document external service integrations
- Example: "Email via SendGrid — API key in SENDGRID_API_KEY env var"
- Example: "File uploads go to S3 bucket defined in AWS_S3_BUCKET"
- Example: "Auth tokens are RS256 signed — public key at /api/auth/.well-known/jwks.json"

---

## Development Setup

```bash
# TODO - document how to run the project locally
# Example:
# cp .env.example .env
# docker-compose up -d    # Start database
# npm install
# npm run db:migrate       # Run migrations
# npm run dev              # Start dev server
```

---

## Jonggrang Workflow

Jonggrang uses a **two-phase planning** flow so humans can review and edit a plan before AI decomposes it into tasks.

### Full workflow

```bash
# Phase 1 — generate a human-readable draft plan
jonggrang plan "add JWT authentication"
# → AI writes .jonggrang/.drafts/<session>/plan.md (high-level, no tasks yet)
# → Interactive options:
#     Approve           → run Phase 2 immediately
#     Edit with AI      → describe changes, AI revises plan, loop back
#     Edit in $EDITOR   → open editor, loop back
#     Save draft        → exit, run "jonggrang approve" later
#     Abort             → discard the draft plan.md

# Resume after accidental close:
jonggrang plan
# → no description → shows list of pending + archived plans
# → pick one → shows plan + interactive options again

# Phase 2 — approve plan → decompose into tasks
jonggrang approve
# → AI reads .jonggrang/.drafts/<session>/plan.md → runs `jonggrang task import` to create tasks
# → plan.md is archived to .jonggrang/.output/features/<featureId>/plan.md

# Execute tasks
jonggrang work
```

### Shorthand options

```bash
# Plan + auto-approve + tasks in one shot (skips human review)
jonggrang plan "add JWT auth" --yes

# Deep mode: 3-phase analysis (discovery + brainstorm + condense) → enriched plan
# Adds Affected Areas, Risks, and Alternatives Considered sections to plan.md
jonggrang plan "add JWT auth" --deep

# Deep mode + auto-approve in one shot
jonggrang plan "add JWT auth" --deep --yes

# Full pipeline: plan → approve → execute in one shot
jonggrang work "add JWT auth" --yes

# Execute existing tasks only (skip pending plan warning)
jonggrang work --ignore-plan
```

### Modifying an approved plan

| Situation | Command |
|-----------|---------|
| Add new scope on top of done work | `jonggrang plan "also add rate limiting"` |
| Change remaining pending work | `jonggrang plan "use Passport.js instead"` |
| Undo completed tasks | Not supported — create new tasks to override |

**Rule: completed tasks are immutable.** They reflect real code. Any correction must be a new task that fixes/replaces the previous implementation.

### Plan file format

```markdown
---
feature: jwt-auth
branch: feat/jwt-auth
work_type: MEDIUM
description: JWT authentication with login, register, refresh
created_at: 2026-04-16T10:30:00Z
---

# Plan: JWT Authentication

## Approach
...

## Phases
1. DB schema — users + refresh_tokens
2. Auth service — register/login/refresh
3. JWT middleware
...

## Key Decisions
- Token storage: httpOnly cookie

## Out of Scope

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mayarid/mayar-cli](https://github.com/mayarid/mayar-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
