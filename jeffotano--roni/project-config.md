---
trigger: always_on
description: This file provides guidance to AI coding agents (Claude Code, GitHub Copilot, Cursor, Windsurf, Codex, Gemini, etc.) when working with code in this repository. Follows the [agents.md](https://agents.md) open standard.
---

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, GitHub Copilot, Cursor, Windsurf, Codex, Gemini, etc.) when working with code in this repository. Follows the [agents.md](https://agents.md) open standard.

## What This Project Is

Roni is an AI coaching companion for Tonal fitness machines. Users connect their Tonal account, and the AI coach reads their training history, strength scores, and workout data to program custom weekly plans. The coach pushes approved workouts directly to Tonal.

## Stack

- **Frontend:** Next.js 16 (App Router), React 19, Tailwind CSS v4, shadcn/ui (Base UI)
- **Backend:** Convex (queries, mutations, actions), @convex-dev/agent for AI coach
- **Language:** TypeScript (strict mode), Zod for runtime validation
- **Testing:** Vitest, @vitest/coverage-v8
- **Formatting:** Prettier (auto-enforced via pre-commit hooks)
- **Package manager:** npm
- **Deployment:** Vercel (web), Convex (backend)

## Conductor Workspaces

This project uses [Conductor](https://conductor.build) for parallel agent development. Each workspace is an isolated git worktree with its own branch.

- **Environment files** (`.env.local`, `.env.sentry-build-plugin`) are symlinked from the repo root via the setup script -- do not copy or create them manually
- **Dev server port**: Use `$CONDUCTOR_PORT` instead of hardcoding 3000. The run script handles this automatically
- **Convex dev**: All workspaces share the same Convex dev deployment, so avoid running schema-altering backend changes in parallel. `npx convex dev` is started by the run script
- **Workspace path**: Available as `$CONDUCTOR_WORKSPACE_PATH`. The repo root is `$CONDUCTOR_ROOT_PATH`
- **Branch per workspace**: Each workspace gets a unique branch. Rename with `git branch -m new-name`

## Development Commands

```bash
# Dev servers (run concurrently in separate terminals)
npx convex dev              # Convex backend with hot reload
npm run dev                 # Next.js dev server (port 3000)

# Verification (run after every code change)
npm run typecheck           # tsc --noEmit

# Testing
npm test                    # all tests once
npx vitest --project backend   # backend tests only (convex/**/*.test.ts)
npx vitest --project frontend  # frontend tests only (src/**/*.test.{ts,tsx})
npx vitest run convex/stats.test.ts  # single test file
npm run test:watch          # watch mode
npm run test:coverage       # with coverage report

# Code quality
npm run lint                # ESLint
npm run format              # Prettier (write)
npm run format:check        # Prettier (check only)
npm run knip                # Dead code detection

# Build
npm run build               # Production build

# Convex
npx convex env set KEY value   # Set backend environment variable
npx convex deploy              # Deploy to production

# Smoke-test schema changes against the dev deployment before pushing.
# Pushes the branch's schema to dev once; fails locally if any existing row
# fails validation under the new shape. Auto-runs via the pre-push hook on
# any push that touches convex/schema.ts or convex/aiUsage.ts.
npm run convex:smoke

# Silence cron jobs on dev (optional, recommended)
npx convex env set DISABLE_CRONS true
```

## Architecture

### Core Data Flow

```
Tonal API --> [encrypted tokens] --> Convex proxy/cache layer --> Convex DB
                                                                     |
                                                                     v
User (chat) --> send message --> AI Coach Agent (Gemini, 31 tools) --> reads context
                                                                     |
                                                          creates workoutPlans (draft)
                                                                     |
                                                          user approves --> push to Tonal API
```

### Backend Domains (`convex/`)

- **`ai/`** -- Coach agent definition, 31 tools (read Tonal data, create/modify workouts, manage goals/injuries), context builder that injects training snapshot as system message, prompt construction
- **`coach/`** -- Programming engine: exercise selection, periodization (Building/Deload/Testing blocks), progressive overload tracking
- **`tonal/`** -- Tonal API integration: OAuth token management (AES-256 encrypted at rest), proxy layer with stale-while-revalidate caching, history sync, movement/workout catalog sync
- **`lib/auth.ts`** -- `getEffectiveUserId()` helper used by all user-facing queries/mutations; thin wrapper over `getAuthUserId`

### AI Observability & Evals

- **Phoenix Cloud** is the canonical AI trace destination. `convex/ai/otel.ts` registers an OpenTelemetry provider via `@arizeai/phoenix-otel` when `PHOENIX_API_KEY` is set; it falls back to a no-op provider otherwise. Both `convex/ai/otel.ts` and `convex/ai/resilience.ts` run under `"use node"` because phoenix-otel pulls `@opentelemetry/sdk-trace-node`. Do not import either module from a V8-runtime file.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JeffOtano/roni](https://github.com/JeffOtano/roni) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
