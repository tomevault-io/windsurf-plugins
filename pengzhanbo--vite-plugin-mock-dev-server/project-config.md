---
trigger: always_on
description: `vite-plugin-mock-dev-server` — a Vite plugin providing an API mock dev server: it intercepts HTTP/WS requests that match user-configured proxy prefixes, matches them against mock files, and responds with mock data. Features include mock-file hot reload, validators, scenes, error simulation, cookies, SSE, WebSocket mocks, request record/replay, CORS, and a build-time generator that emits a standalone mock server.
---

# Repository Guidelines

## Project Overview

`vite-plugin-mock-dev-server` — a Vite plugin providing an API mock dev server: it intercepts HTTP/WS requests that match user-configured proxy prefixes, matches them against mock files, and responds with mock data. Features include mock-file hot reload, validators, scenes, error simulation, cookies, SSE, WebSocket mocks, request record/replay, CORS, and a build-time generator that emits a standalone mock server.

Monorepo (pnpm workspaces): the plugin package `vite-plugin-mock-dev-server/` (only published artifact), an `example/` Vite app, and `docs/` (VitePress). All package manifests, `pnpm-workspace.yaml`, and configs live at the repo root.

## Architecture & Data Flow

Request pipeline: **configure → compile → match → respond**

1. **Configure**: `mockDevServerPlugin(options)` (`vite-plugin-mock-dev-server/src/core/plugin.ts`) returns `[serverPlugin, buildPlugin?]`. `config()` strips `wsPrefix` entries from `server.proxy`, wires request-body recovery, sets up the recorder; `configResolved` merges user options with `server.proxy` keys via `resolvePluginOptions` (`core/options.ts`).
2. **Compile**: `configureServer` → `initMockMiddlewares` (`core/init.ts`) → `Compiler` (`compiler/compiler.ts`), an `EventEmitter` that globs `include` patterns (default `**/*.mock.{js,ts,cjs,mjs,json,json5}` under `dir` default `mock/`), bundles each mock file (`compile.ts` picks **rolldown if installed, else esbuild**; Vite `define`/alias injection; `vite-plugin-mock-dev-server` imports renamed to the `/helper` subpath), loads via temp-file dynamic `import()`, then normalizes exports through `processRawData` (stamps `__filepath__`) and `processMockData` (groups by pathname, embeds URL query into validators, sorts by validator weight). Chokidar watches mock files + their deps; updates emit `mock:update-end` → optional Vite full-reload and WS mock restart.
3. **Match**: `createMockMiddleware` (`mockHttp/middleware.ts`) per request: `urlParse` → proxy-prefix gate (`doesProxyContextMatchUrl`) → `matchingWeight` sorts candidate URL rules (static < dynamic < wildcard priority; `priority.global`/`priority.special` overrides) → body parse (`co-body`/formidable, raw body cached for proxy recovery) → `findMockData` (`mockHttp/matcher.ts`): method filter (default `['GET','POST']`), scene match (`activeScene` option or `X-Mock-Scene` header), `isPathMatch`, then validator (function or object-subset). First hit wins. Miss → replay from recordings, else record + `next()`.
4. **Respond**: CORS (matched requests only) → attach request extras (`query`, `params`, `body`, `getCookie`, …) → optional error simulation (`error.probability`) → status/headers/cookies → body: value, sync/async `body` fn, or `response(req, res, next)` middleware; `delay` number or `[min, max]`. Failures → `logger.error` + 500.
5. **WebSocket**: `mockWebsocket/server.ts` listens `upgrade`; per-URL `WebSocketServer({ noServer: true })`; mock `setup(wss, { onCleanup })`; HMR restarts affected sockets.
6. **Record/Replay**: `recorder/` hooks `proxyRes` on non-ws proxies; recordings stored as pretty-printed JSON (`kebabCase(pathname).json`), replayed as synthetic mock items with decompression (gzip/br/zstd).
7. **Build**: `buildPlugin.buildEnd` → `build/generate.ts` emits `mockServer/` (standalone connect server + bundled mock data + package.json).

## Key Directories

| Path                                             | Purpose                                                                                                                                 |
| ------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
| `vite-plugin-mock-dev-server/src/`               | Plugin source (ESM, `.js` import suffixes, per-dir `index.ts` barrels)                                                                  |
| `vite-plugin-mock-dev-server/src/core/`          | Plugin lifecycle: `plugin.ts`, `options.ts`, `init.ts`, `define.ts` (Vite define fork), `logger.ts`                                     |
| `vite-plugin-mock-dev-server/src/compiler/`      | Mock-file bundling/loading/watch: `compiler.ts`, `esbuild.ts`, `rolldown.ts`, `compile.ts`, `processData.ts`                            |
| `vite-plugin-mock-dev-server/src/mockHttp/`      | HTTP request pipeline: `middleware.ts`, `request.ts`, `response.ts`, `matcher.ts`, `matchingWeight.ts`, `cors.ts`, `requestRecovery.ts` |
| `vite-plugin-mock-dev-server/src/mockWebsocket/` | WS mock server (`server.ts`)                                                                                                            |
| `vite-plugin-mock-dev-server/src/helpers/`       | Public helpers: `defineMock.ts`, `defineMockData.ts`, `createSSEStream.ts`                                                              |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pengzhanbo/vite-plugin-mock-dev-server](https://github.com/pengzhanbo/vite-plugin-mock-dev-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
