---
trigger: always_on
description: This file provides guidance to the agent when working with code in this repository.
---

# AGENTS.md

This file provides guidance to the agent when working with code in this repository.

## Project Overview

TeamCopilot is an open-source multi-user AI agent platform for teams that want shared agent capabilities without giving up control, visibility, or operational guardrails.

At a high level, TeamCopilot combines:
- a web UI for chatting with an agent remotely
- a shared filesystem-backed workspace of custom skills and workflows
- approval and permission controls for risky or reusable automation
- auditability for chat sessions and file changes
- team-oriented secret management for agent-assisted execution

## Repository Structure

This is a monorepo (without workspaces) containing:
- **Backend** (root): Express.js + TypeScript + Prisma + SQLite — lives in `src/`
- **Frontend** (`frontend/`): React 19 + TypeScript + Vite — separate `package.json`

The backend serves the built frontend from `frontend/dist/` as static files with SPA fallback routing.

## Commands

### Backend (run from repo root)
```bash
npm run dev              # Dev server with auto-reload (ts-node-dev)
npm run build            # Compile TypeScript to dist/
npm start                # Build + run (tsc && node dist/index.js)
```

### Frontend (run from frontend/)
```bash
npm run dev              # Vite dev server with HMR
npm run build            # TypeScript check + Vite production build
npm run lint             # ESLint
npm run preview          # Preview production build
```

### Database (Prisma, run from repo root)
```bash
npm run prisma:migrate:dev -- --name <description>   # Create and apply migration
npm run prisma:migrate:reset                          # Reset database (deletes all data)
npm run prisma:migrate:deploy                         # Deploy migrations
npm run prisma:studio                                 # Visual database browser
```

### Initial Setup
```bash
npm install
cd frontend && npm install && cd ..
cp .env.example .env     # Then fill in values
cd frontend && npm run build && cd ..
```

## Architecture

### Backend API Pattern

All authenticated/versioned API routes use the `apiHandler` wrapper from `src/utils.ts`:

```typescript
apiRouter.get("/endpoint", apiHandler(async (req, res) => {
    // req.userId, req.email, req.name available
}, true));
```

`apiHandler` handles: API version validation from the URL path, JWT token verification from `Authorization: Bearer` header, and user lookup from Prisma. The `CustomRequest` type extends Express Request with `version`, `userId`, `email`, and `name`.

### Authentication Flow

Email/password auth → JWT tokens (365-day expiry). The flow:
1. `POST /api/auth/signup` — creates user with bcrypt-hashed password, returns JWT
2. `POST /api/auth/signin` — validates credentials, returns JWT
3. Frontend stores token in localStorage and sends `Authorization: Bearer {token}` on subsequent requests
4. Password reset via CLI: `npm run reset-password -- user@example.com` sets a temporary password

### Route Structure

All API routes are mounted under `/api` via `apiRouter`.

Non-API `GET *` requests serve `frontend/dist/index.html` for client-side routing.

`src/index.ts` also exports `createApp()` so route-level tests can mount the Express app in-process without starting a separate server.

### Database

SQLite via Prisma ORM. Schema is at `prisma/schema.prisma`. The database now stores much more than just auth metadata, including users, permissions, approvals, chat/session data, and secret-management tables such as `user_secrets` and `global_secrets`. SQLite uses WAL mode for concurrency (configured in `src/prisma/client.ts`).

### Shared Workspace Model

TeamCopilot is filesystem-first:
- workflows live in `workflows/<slug>/`
- custom skills live in `.agents/skills/<slug>/`
- the filesystem contents are the source of truth for those resources

This means most resource creation, editing, approval diffs, and runtime behavior are driven by real files on disk rather than opaque database blobs.

### Secrets Model

TeamCopilot has two secret scopes:
- user secrets: private to a specific user
- global secrets: shared across the team and managed by engineers

Resolution order is:
1. current user's secret
2. global secret
3. missing

Important implementation details:
- workflows declare required secret keys in `workflow.json` under `required_secrets`
- workflow runs receive resolved secrets as environment variables at runtime
- skills declare required secret keys in `SKILL.md` frontmatter
- skills keep placeholder references like `{{SECRET:OPENAI_API_KEY}}` in file content rather than storing raw secret values
- agent-facing secret usage in shell goes through a secret-proxy model; plaintext secret values are not included in the LLM-visible context
- `.env` files are still redacted in editor/file APIs, but AI-authored workflows should rely on declared runtime secrets rather than workflow-local `.env` files

### Approvals And Runtime Boundaries

- Workflows and custom skills can require engineer approval before they are usable.
- Workflow execution must go through the platform runtime rather than direct shell execution of `run.py`.
- Approval diffs and chat session file diffs are part of the product’s audit trail and are intentionally inspectable.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rishabhpoddar/teamcopilot](https://github.com/rishabhpoddar/teamcopilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
