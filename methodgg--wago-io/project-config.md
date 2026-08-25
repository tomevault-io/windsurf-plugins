---
trigger: always_on
description: This repository contains the current Wago v3 web application. Make small, focused changes. Preserve current behavior unless the task requires a behavior change. Do not start a broad framework migration as part of an unrelated task.
---

# Wago.io agent guide

## Purpose

This repository contains the current Wago v3 web application. Make small, focused changes. Preserve current behavior unless the task requires a behavior change. Do not start a broad framework migration as part of an unrelated task.

## Start each task

1. Read this file and the files that you will change.
2. Read one nearby implementation that does similar work.
3. Check `git status --short` and keep unrelated user changes intact.
4. Identify all affected parts of the system. The backend imports some frontend files directly.
5. State any validation limit. This project does not have one reliable repository-wide test command.

## Runtime and package layout

- Use Node.js 18.20.4. Both application packages pin this version with Volta, and the development container uses Node.js 18.
- Use npm and the package lockfile in each directory. There is no root package or workspace.
- Run package commands from the package directory that owns the lockfile.
- `backend/` is the Fastify 3 API, background worker, and data layer.
- `frontend/` is the Vue 2 single-page application and Webpack 5 build.
- `scripts/` contains operational scripts with their own dependencies and lockfile.
- If you add Python tooling, use `uv`.

Install only the dependencies that a task needs:

```sh
(cd backend && npm ci)
(cd frontend && npm ci)
(cd scripts && npm ci) # Only for a scripts/ task.
```

Do not run npm from the repository root. Update only the lockfile that belongs to the changed package.

## Architecture map

### Backend

- `backend/server.js` is the composition root. It generates and serves the OpenAPI contract, creates Fastify, connects Redis and MongoDB, registers global hooks and route plug-ins, loads models and import handlers, and starts BullMQ workers and repeat jobs.
- Run backend processes with `backend/` as the current directory. Several modules use paths such as `./api/models` and `./api/lua/wago.lua`.
- `backend/api/services/` contains Fastify route plug-ins. Existing plug-ins use the Fastify 3 callback form and call `next()` after route registration.
- `backend/middlewares/` contains global request hooks. Their registration order in `server.js` is part of request behavior.
- `backend/api/models/` contains Mongoose 5 models.
- `backend/api/helpers/` contains search, import, Lua, image, integration, and background-task logic.
- `backend/api/helpers/encode-decode/` contains the auto-loaded import adapters. Follow its `Readme.md` and a current adapter when you add an import type.
- `backend/tools/generate-openapi.js` is the source for the WeakAuras API contract used by Wago App. It writes the tracked `public/openapi.json` file.
- Much of the backend depends on globals initialized by `server.js`, including models, Redis clients, queues, configuration, categories, and import adapters. Do not assume that a module is safe to load in isolation.

### Frontend

- `frontend/src/main.js` is the browser composition root. It owns Vuex state, router hooks, HTTP behavior, authentication headers, sockets, i18n, global components, and application startup.
- `frontend/src/router.js` is the route table. It uses lazy-loaded Vue 2 components. Keep the `/:wagoID` catch-all routes last.
- `frontend/src/components/core/` contains feature and page components.
- `frontend/src/components/UI/` contains shared UI components.
- `frontend/src/components/libs/` contains shared registries and browser-side helpers.
- Use Vue 2 Options API and the existing Vuex, Vue Router 3, and global plug-in patterns. Do not introduce Vue 3 or Composition API patterns without an explicit migration task.
- The development frontend uses `http://localhost:3030` for the API and `ws://localhost:3030/ws` for sockets. Webpack serves the frontend on port 8080.

### Cross-application seams

Treat these frontend files as shared backend modules:

- `frontend/src/components/libs/categories2.js` is the main category registry used by both applications and several import and task helpers.
- `frontend/src/components/libs/categories.js` is still used by `backend/ProcessCategoryRelevancy.js`. Treat that standalone process as a separate compatibility path.
- `frontend/src/components/libs/addons.js` initializes category and add-on data in both applications.
- `frontend/src/router.js` is loaded by `backend/api/services/wago.js`. Top-level frontend routes also reserve custom Wago slugs.
- `i18nLocaleConfig.js` and `frontend/static/i18n/` are used by both applications.

When you change a route, add-on, category, expansion, domain, or translation key, search for its use in both `backend/` and `frontend/`. A frontend-only diff can still change backend behavior.

## Local configuration and external effects

- `backend/config.js` is ignored and can contain secrets. Never commit it or print secret values.
- `backend/config.js.sample` is a starting reference, not proof of a complete working configuration. The backend also needs MongoDB, Redis, search services, LuaJIT, and task-specific external credentials.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [methodgg/wago.io](https://github.com/methodgg/wago.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
