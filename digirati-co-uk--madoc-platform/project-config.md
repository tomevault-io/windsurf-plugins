---
trigger: always_on
description: This is the default instruction file for agents working in `services/madoc-ts`.
---

# AGENTS.md (madoc-ts)

This is the default instruction file for agents working in `services/madoc-ts`.
Use it as the cross-cutting guide for how Madoc works, how to change it safely, and how to verify changes locally.

## Philosophy

- No area is off-limits — some just need more care
- Understand before modifying — read code, tests, trace calls
- Ask when uncertain — ambiguity warrants a question
- Flag sensitive changes — tell the user when touching elevated care zones

## Scope and skill usage

- Keep README/docs human-focused; keep this file agent-focused.
- This file is self-contained, but skills still apply. If a task matches a skill, use it for subsystem-specific workflow and constraints.
- Do not duplicate deep skill instructions here; use this file for repository-wide rules and architecture context.

## Skill upkeep

- If you learn a reusable workflow, recurring pitfall, or domain rule that is not captured in an existing skill, create a new skill.
- If you modify behavior in an area already covered by a skill, update that skill in the same change so instructions stay accurate.
- If you find a mistake or outdated guidance in a skill, fix it immediately.
- Keep skill changes concise and practical; avoid duplicating this AGENTS file inside skills.

## Madoc runtime model (high-level)

- `madoc-ts` runs as multiple PM2 apps in one container: `server`, `auth`, `queue`, `scheduler`.
- Entrypoints are `entrypoint/server.cjs`, `entrypoint/auth.cjs`, `entrypoint/producer.cjs`, `entrypoint/scheduler.cjs`.
- These load built bundles from `dist/server`, `dist/auth`, `dist/producer`, `dist/scheduler`.
- Startup and middleware wiring live in `src/app.ts`.
- Route registration is centralized in `src/router.ts` using `TypedRouter`.
- Site routing tree is in `src/frontend/site/routes.tsx`.
- Page-block and slot context behavior live in `src/extensions/page-blocks/extension.ts`.

## Request pipeline and auth behavior

- Core middleware order matters: parse/set JWT, site state, static page handling, then router.
- `TypedRouter` applies middleware by HTTP method:
- `GET`: `parseJwt` + `siteState` unless route is explicitly `isPublic`.
- `POST`/`PUT`/`PATCH`: `parseJwt` + `requestBody(schemaName)`.
- Unauthorized/scope failures often intentionally return `NotFound`; preserve this behavior unless a task explicitly changes auth semantics.
- When creating scoped API clients in routes (`api.asUser(...)`), add them to `context.disposableApis` so `dispose-apis` can clean up.

## Frontend rendering and data-loading model

- Site/admin are SSR-first; dev mode injects Vite client from port `3088`.
- Route loaders use `createUniversalComponent(..., { getKey, getData, hooks, noSsr })`.
- Keep loader keys deterministic and stable (same params/query -> same key) to avoid hydration/query-cache drift.
- Prefer `useData` / `useStaticData` and `ApiClient` extension methods over ad-hoc `fetch`.
- SSR-safe rule: do not use `window`/`document`/browser APIs without explicit guard (`api.getIsServer()` or equivalent).
- Build links with `createLink`/`useRelativeLinks`; avoid hardcoded URL strings that lose context.

## Frontend context model (critical)

- Site pages run in resource context. A page can have zero or more of `projectId`, `collectionId`, `manifestId`, `canvasId`, `taskId`.
- Context is merged from URL params and slot context in `src/frontend/site/hooks/use-route-context.ts`.
- The special `/model` route exists on manifest/canvas flows and must be handled explicitly when adding route-aware logic.
- `PageLoader` builds `EditorialContext` and provides slot context (`project`, `collection`, `manifest`, `canvas`) for page blocks.

## Page blocks, slots, and templates

- Page blocks are context-aware and can render on many pages when required context matches.
- When adding/changing a block, define `requiredContext` / `anyContext` correctly and keep behavior safe when optional context is missing.
- Default site pages use slot-driven blocks, but most pages are customizable.
- New optional features should be implemented as blocks/slot config where possible, not hardcoded page-only behavior.
- Project templates can inject slot mappings at project creation (see `create-new-project` flow), so slot compatibility matters for custom templates.
- If feature behavior depends on page location, test it in both direct routes and slot-resolved routes.

## API and route change checklist

- If a new backend endpoint is needed, wire it fully:
- handler in `src/routes/...`
- route entry in `src/router.ts`
- request schema in `schemas/` and `schemaName` in route options when request body is validated
- `ApiClient` method in `src/gateway/api.ts` when frontend/server consumers need it
- Use route names and `context.routes.url(...)` for server-generated links (emails, redirects, callbacks) instead of hardcoding URL fragments.
- Keep route permissions explicit with `userWithScope` / `onlyGlobalAdmin` and avoid leaking partial access paths.

## Styling and component conventions

- Tailwind is the forward path.
- Do not add new `styled-components` code for feature work.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [digirati-co-uk/madoc-platform](https://github.com/digirati-co-uk/madoc-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
