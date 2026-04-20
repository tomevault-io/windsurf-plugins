---
trigger: always_on
description: AlphaClaw is the ops and setup layer around OpenClaw. It provides a browser-based setup UI, gateway lifecycle management, watchdog recovery flows, and integrations (for example Telegram, Discord, Google Workspace, and webhooks) so users can operate OpenClaw without manual server intervention.
---

## Project Overview

### AlphaClaw Project Context

AlphaClaw is the ops and setup layer around OpenClaw. It provides a browser-based setup UI, gateway lifecycle management, watchdog recovery flows, and integrations (for example Telegram, Discord, Google Workspace, and webhooks) so users can operate OpenClaw without manual server intervention.

### Understanding OpenClaw

If you need to understand the internals of OpenClaw, you can inspect the code at `~/Projects/openclaw/src`

### Architecture At A Glance

- `bin/alphaclaw.js`: CLI entrypoint and lifecycle command surface.
- `lib/server`: Express server, authenticated setup APIs, watchdog APIs, channel integrations, and proxying to the OpenClaw gateway.
- `lib/public`: Setup UI frontend (component-driven tabs and flows for providers, envars, watchdog, webhooks, and onboarding).
- `lib/setup`: Prompt hardening templates and setup-related assets injected into agent/system behavior.

Runtime model:

1. AlphaClaw server starts and manages OpenClaw as a child process.
2. Setup UI calls AlphaClaw APIs for configuration and operations.
3. AlphaClaw proxies gateway traffic and handles watchdog monitoring/repair.

### Key Technologies

- Node.js 22.14+ runtime.
- Express-based HTTP API server.
- `http-proxy` for gateway proxy behavior.
- OpenClaw CLI/gateway process orchestration.
- Preact + `htm` frontend patterns for Setup UI components.
- Vitest + Supertest for server and route testing.

## Coding Conventions

### Change Patterns

- Keep edits targeted and production-safe; favor small, reviewable changes.
- Preserve existing behavior unless the task explicitly requires behavior changes.
- Follow existing UI conventions and shared components for consistency.
- Reuse existing server route and state patterns before introducing new abstractions.
- Update tests when behavior changes in routes, watchdog flows, or setup state.
- Before running tests in a fresh checkout, run `npm install` so `vitest` (devDependency) is available for `npm test`.

### Code Structure

- Avoid monolithic implementation files for new features. For new UI areas and new API areas, start with a decomposed structure (focused components/hooks/utilities for UI; focused route modules/services/helpers for server) rather than building one large file first and splitting later.
- When adding a new feature area, follow the existing project patterns from day one (for example feature folders with `index.js` plus `use-*` hooks in UI, and route + service separation on server) so code stays maintainable as the feature grows.
- When continuing to build on a file that is growing large or accumulating unrelated concerns, stop and decompose it before adding more code rather than letting it drift into a monolith.

### Networking and Fetching

- Prefer the shared cache primitives in `lib/public/js/lib/api-cache.js` for backend reads:
  - `cachedFetch(...)` for imperative fetch paths.
  - `getCached(...)` / `setCached(...)` / `invalidateCache(...)` for cache lifecycle.
- For component-level read requests, prefer `useCachedFetch` from `lib/public/js/hooks/use-cached-fetch.js` over ad-hoc `useEffect(() => fetchX())` mount loads.
- Treat the API URL (including query params) as the canonical cache key for GET-style payloads.
- Keep cache in-memory for fast tab switches; do not add persistent storage caching unless explicitly required by product behavior.
- Do not keep route panes mounted via `display:none` just to preserve data. Prefer conditional rendering + cache-backed remounts.
- Use `usePolling` for recurring refreshes and always pass a stable `cacheKey` when poll results should hydrate remounts.
- Keep `pauseWhenHidden` behavior enabled for polling unless a specific flow requires background polling while the browser tab is hidden.
- Tune polling intervals conservatively; avoid 1-2s polling unless there is a clear real-time requirement.
- For app-shell status streams, prefer SSE (`/api/events/status`) where available and keep polling as fallback behavior.
- After write/mutation APIs (POST/PUT/DELETE), refresh or invalidate relevant cached keys so the UI does not show stale data.

### OpenClaw Config Access

- When reading `openclaw.json` in server code, use the shared helper in `lib/server/openclaw-config.js` (`readOpenclawConfig`) instead of ad-hoc `JSON.parse(fs.readFileSync(...))` blocks.

### Where To Put Agent Guidance

- **This file (`AGENTS.md`):** Project-level guidance for coding agents working on the AlphaClaw codebase — architecture, conventions, release flow, UI patterns, etc.
- **`lib/setup/core-prompts/AGENTS.md`:** Runtime prompt injected into the OpenClaw agent's system prompt. Only write there when the guidance is meant for the deployed agent's behavior, not for coding on this project.

## Operations

### Release Flow (Beta -> Production)

Use this release flow when promoting tested beta builds to production:

1. Ensure `main` is clean and synced, and tests pass.
2. Publish beta iterations as needed:
   - `npm version prerelease --preid=beta`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrysb/alphaclaw](https://github.com/chrysb/alphaclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
