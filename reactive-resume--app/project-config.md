---
trigger: always_on
description: <!-- intent-skills:start -->
---

<!-- intent-skills:start -->
## Skill Loading

Before editing files for a substantial task:
- Run `pnpm dlx @tanstack/intent@latest list` from the workspace root to see available local skills.
- If a listed skill matches the task, run `pnpm dlx @tanstack/intent@latest load <package>#<skill>` before changing files.
- Use the loaded `SKILL.md` guidance while making the change.
- Monorepos: when working across packages, run the skill check from the workspace root and prefer the local skill for the package being changed.
- Multiple matches: prefer the most specific local skill for the package or concern you are changing; load additional skills only when the task spans multiple packages or concerns.
<!-- intent-skills:end -->

<!-- caveman-begin -->
Respond terse like smart caveman. All technical substance stay. Only fluff die.

Rules:
- Drop: articles (a/an/the), filler (just/really/basically), pleasantries, hedging
- Fragments OK. Short synonyms. Technical terms exact. Code unchanged.
- Pattern: [thing] [action] [reason]. [next step].
- Not: "Sure! I'd be happy to help you with that."
- Yes: "Bug in auth middleware. Fix:"

Switch level: /caveman lite|full|ultra|wenyan-lite|wenyan-full|wenyan-ultra
Stop: "stop caveman" or "normal mode"

Auto-Clarity: drop caveman for security warnings, irreversible actions, user confused. Resume after.

Boundaries: code/commits/PRs written normal.
<!-- caveman-end -->

## Agent skills

- Issues and specs: GitHub Issues for `reactive-resume/reactive-resume`. See `docs/agents/issue-tracker.md`.
- Domain docs use a multi-context layout. See `docs/agents/domain.md`.

## Overview

Reactive Resume is a pnpm monorepo (Turborepo) with two deployable apps: `apps/web` (TanStack Start / React 19 / Vite) and `apps/server` (Hono / Node.js). The production Docker image runs a single Node.js process on port 3000; `apps/server` mounts the API/auth/MCP/static routes and serves the built web app.

Internal packages are source-consumed through `package.json` export maps pointing at `src` files. Do not assume package-local `dist` output exists unless a package explicitly adds it.

Prerequisites: **Node.js 24** (pinned in `.nvmrc`; matches Dockerfile `ARG NODE_VERSION=24`), **pnpm 12.3.4** (pinned by `packageManager` in the root `package.json`; pnpm self-manages to it, so any recent pnpm can bootstrap — the Dockerfile's `ARG PNPM_VERSION` only picks the base image) ([install guide](https://pnpm.io/installation)), and **Docker** for PostgreSQL (`sudo dockerd &` if the daemon isn't running).

## Ownership map

Where each concern lives, and where new code for it goes:

| Area | Owner |
|------|-------|
| Web routes, loaders, user-facing workflows | `apps/web/src/routes`, `apps/web/src/features` (file-based; never hand-edit `routeTree.gen.ts`) |
| Server HTTP routes/adapters, startup checks, static handlers, MCP transport, OpenAPI/well-known | `apps/server/src/{http,rpc,mcp,openapi,static,startup}` |
| Authenticated API contracts + business logic | `packages/api/src/features/*` (oRPC routers, DTOs, rate limiting; aggregated at `@reactive-resume/api/routers` for `/api/rpc`) |
| Auth | `packages/auth` (Better Auth config/helpers/types; `apps/server/src/http/auth.ts` delegates to `auth.handler`) |
| DB client + schema | `packages/db` (Drizzle; migrations at repo root `migrations/`) |
| Server env validation | `packages/env` (auto-loads root `.env`) |
| Resume/page/template Zod schemas | `packages/schema` |
| Pure resume-domain behavior (no DB/HTTP/DOM/renderer deps) | `packages/resume` (JSON Patch helpers, social-network icons) |
| Resume PDF rendering | `packages/pdf` (React PDF document, font registration, template primitives, browser/server adapters) |
| PDF.js viewer/canvas UI | `apps/web/src/features/resume` — never in `packages/pdf` |
| DOCX export | `packages/docx` |
| MCP tools/prompts/resources/server-card | `packages/mcp` |
| Generic UI primitives + hooks | `packages/ui` (Base UI/shadcn-style); workflow-specific UI stays in the owning web feature |
| Focused support surfaces | `packages/fonts`, `packages/email`, `packages/import`, `packages/ai`, `packages/utils`, `packages/config` — prefer existing exports over cross-package shortcuts |
| Dev-only scripts | `tooling/`, not `packages/`, so packages only hold runtime-bundled code |

Narrow cross-cutting helpers go in `packages/utils` only after checking no domain package is a better owner. Specifically: resume JSON Patch behavior belongs in `@reactive-resume/resume/patch` and DOCX builders in `@reactive-resume/docx` — not in `@reactive-resume/utils`.

## Web app conventions

- `apps/web/src/router.tsx` initializes router context with `queryClient`, `orpc`, `theme`, `locale`, `session`, and `flags`. Reuse route context instead of refetching these ad hoc.
- Builder shell: `apps/web/src/routes/builder/$resumeId`. Its nested preview route is client-only (`ssr: false`); the public resume route `apps/web/src/routes/$username/$slug.tsx` uses `ssr: "data-only"`.
- Browser-only preview code: `apps/web/src/features/resume/preview`. Public PDF viewer: `apps/web/src/features/resume/public`. Keep PDF.js/canvas/browser APIs out of SSR paths.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reactive-resume/app](https://github.com/reactive-resume/app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
