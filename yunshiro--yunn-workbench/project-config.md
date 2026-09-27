---
trigger: always_on
description: Yunn Workbench is a local-first content research application distributed as an npm CLI. It combines RSS ingestion, a SQLite-backed Express API, a Pi-powered AI task queue, and a React/Vite UI. The supported runtime is Node.js 24 or newer because the server uses `node:sqlite`.
---

# AGENTS.md

## Repository Overview

Yunn Workbench is a local-first content research application distributed as an npm CLI. It combines RSS ingestion, a SQLite-backed Express API, a Pi-powered AI task queue, and a React/Vite UI. The supported runtime is Node.js 24 or newer because the server uses `node:sqlite`.

The normal data flow is:

```text
React UI -> REST API / SSE -> Express -> SQLite
                                      -> RSS feeds
                                      -> Pi agent / configured model provider
```

The application has no authentication layer and is designed to listen on the loopback interface. Do not make it publicly reachable without adding an explicit security model.

## Repository Layout

- `cli/yworkbench.js`: published `yworkbench` executable. It validates CLI arguments, sets runtime environment variables, imports the built server, and opens the browser.
- `server/src/index.ts`: server lifecycle, static web hosting, scheduled RSS refresh, and retention cleanup.
- `server/src/http.ts`: all `/api` routes, request validation, error responses, and SSE broadcasting.
- `server/src/db.ts`: SQLite schema, forward migrations, row mappers, and persistence functions.
- `server/src/rss.ts`: RSS probing, fetching, normalization, and item ingestion.
- `server/src/agent.ts`: in-process task queue, Pi sessions, research conversations, task state transitions, and emitted events.
- `server/src/agent-auth.ts`: model profiles, provider credentials, active-model selection, and connection tests.
- `server/src/prompts.ts`: editable prompt templates and task prompt construction.
- `server/src/structured-output.ts`: parsing and validation of model JSON output.
- `server/src/research-tools.ts`: local read-only tools exposed to research sessions.
- `web/src/views/`: page-level React components.
- `web/src/components/`: shared UI and feature components.
- `web/src/lib/api.ts`: browser REST client; keep it aligned with server routes.
- `web/src/lib/sse.ts`: the single global EventSource connection and supported event names.
- `web/src/lib/types.ts`: client-side API/domain types.
- `web/src/styles/global.css`: global design tokens and component styles.
- `docs/`: design and implementation notes; these are not runtime inputs.
- `server/dist/` and `web/dist/`: generated release artifacts. Never hand-edit them.

## Setup and Commands

Run commands from the repository root unless noted otherwise.

- `npm run install:all`: install root, server, and web dependencies.
- `npm run dev:server`: run the API with `tsx watch` on `127.0.0.1:3001` by default.
- `npm run dev:web`: run Vite on port `5173`; `/api` and SSE traffic are proxied to port `3001`.
- `npm run build:web`: type-check and build the React application.
- `npm run build:server`: compile `server/src` into `server/dist`.
- `npm run build`: build both packages in release order.
- `npm start`: run the CLI against built assets. This is not a source-mode command; build first.
- `cd server && npm exec -- tsc --noEmit`: quick server-only type-check.
- `node cli/yworkbench.js --help`: smoke-test CLI argument loading after a build.

For development, run the server and web commands in separate terminals. If the backend port changes, also account for the fixed Vite proxy target in `web/vite.config.ts`.

## Verification Expectations

There is currently no automated test runner or lint script. For every change, run the narrowest relevant checks and report exactly what ran.

- Server-only change: `cd server && npm exec -- tsc --noEmit`.
- Web-only change: `npm run build:web`.
- Cross-package, CLI, schema, or release change: `npm run build` and `node cli/yworkbench.js --help`.
- API or UI behavior change: manually exercise the affected flow in addition to compiling it.
- SSE or agent-task change: verify initial REST state and subsequent live event updates; the UI relies on both.

If tests are introduced, use `*.test.ts` or `*.test.tsx`, add an explicit `npm test` script to the owning package, and never point tests at the user's default data directory.

## Coding Conventions

- Use strict TypeScript, ES modules, two-space indentation, double quotes, semicolons, and trailing commas where the surrounding code uses them.
- Use `PascalCase` for React components and exported types, `camelCase` for functions and variables, and descriptive `UPPER_SNAKE_CASE` for constants.
- Preserve the existing BEM-like CSS naming style, such as `nav__item--active`.
- Server-side relative imports must include the `.js` extension so emitted ESM runs correctly.
- Prefer small functions and existing local patterns over new abstractions or dependencies.
- No formatter or linter is configured. Avoid unrelated formatting churn.
- User-facing text is primarily Chinese; match the language and tone of the surrounding UI.
- Do not edit generated files, `*.tsbuildinfo`, or lockfiles unless the corresponding source or dependency change requires it.

## Cross-Layer Change Rules

### REST API and shared data

When adding or changing an endpoint, update all affected layers together:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yunshiro/yunn-workbench](https://github.com/Yunshiro/yunn-workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
