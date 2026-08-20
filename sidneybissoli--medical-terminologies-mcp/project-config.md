---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run build           # esbuild bundle: src/index.ts -> dist/index.js (Node, ESM, node20)
npm run build:worker-lib # esbuild bundle: src/worker-lib.ts -> dist/worker-lib.js (the Worker's view of the package)
npm run build:all       # both bundles
npm start               # node dist/index.js (runs the MCP server over stdio)
npm run dev             # build + start (stdio)
npm test                # vitest run (skips integration tests by default)
npm run test:watch      # vitest in watch mode
npm run typecheck       # tsc --noEmit (strict; not invoked by `npm run build`)

node scripts/smoke-mcp.mjs           # smoke against the hosted Worker (initialize/tools/list/calls)
node scripts/smoke-mcp.mjs --stdio   # same smoke over local STDIO (requires npm run build)
```

The Cloudflare Worker (`worker/`, not published to npm) is an instance of the
portfolio's Fase 0 hosting template (`mcp-br-commons/templates/cloudflare-worker`)
and has its own scripts, run from inside `worker/` (each first rebuilds
`dist/worker-lib.js` at the root):

```bash
cd worker && npm run dev        # wrangler dev — local HTTP transport on :8787
cd worker && npm run deploy     # wrangler deploy
cd worker && npm run typecheck  # tsc --noEmit
cd worker && npm test           # vitest — auth, rate limit, usage, status, surface
```

The Worker intentionally does **not** list `@modelcontextprotocol/server` or `zod`
in its own deps — they resolve from the parent package's `node_modules` so there is
a single SDK copy (avoids duplicate-instance type clashes with `registerAll`).
`worker/.npmrc` pins `legacy-peer-deps=true` so npm does not install a second copy
to satisfy the `agents` package's hard peer ranges.

Run a single test: `npx vitest run src/utils/cache.test.ts` (or `-t '<name pattern>'` for a single case).

The package is published to npm with `"bin": { "medical-terminologies-mcp": "dist/index.js" }`, so consumers can launch the stdio server via `npx medical-terminologies-mcp` without cloning. `prepublishOnly` runs `npm run build` automatically before `npm publish` so the bundle is fresh in every release.

Run integration tests against live APIs: `INTEGRATION_TESTS=1 npm test`. They live in `src/integration/` and skip by default. WHO and SNOMED integration tests skip cleanly when their respective creds/flags (`WHO_CLIENT_ID`/`WHO_CLIENT_SECRET`, `ENABLE_SNOMED_TOOLS`/`SNOMED_BASE_URL`) are absent. The daily cron CI workflow at `.github/workflows/integration.yml` runs them and surfaces upstream API drift close to when it happens.

The build is two `esbuild` invocations sharing the same source tree. The Node build (`dist/index.js`) targets `node20` and externalizes ALL npm dependencies (`--packages=external` — they resolve from runtime `node_modules` via the `dependencies` field), so the published bundle contains only project code plus the inlined JSON datasets. Externalizing is deliberate: consumers npm-install the deps anyway, and it keeps the published artifact auditable — supply chain scanners (e.g. Socket.dev) attribute each dependency's capabilities (pino's fs/env access, etc.) to that package instead of to this one. The worker-lib build (`dist/worker-lib.js`, from `src/worker-lib.ts`) targets `es2022`/`workerd` conditions, aliases bare Node imports to their `node:` namespaced equivalents, and inlines all app code + datasets — but keeps `@modelcontextprotocol/*` and `zod` EXTERNAL so the Worker, the `agents` package and this lib share the single SDK copy in the parent `node_modules`. Both builds use `tree-shaking=false` — see "Tool registration" below.

Both entry points import `package.json` directly (`resolveJsonModule: true`) so `SERVER_INFO.version` stays in sync with `package.json` — bump the version there only.

To exercise the stdio server interactively: `npx @modelcontextprotocol/inspector node dist/index.js`. To exercise the Worker locally: `cd worker && npm run dev` then `npx @modelcontextprotocol/inspector --transport streamable-http --server-url http://localhost:8787/mcp`.

## Runtime requirements

- Node.js >= 20 (ESM, top-level imports with side effects).
- `WHO_CLIENT_ID` / `WHO_CLIENT_SECRET` are required only for the 5 ICD-11 tools (OAuth2 client credentials). The server will still start without them; ICD-11 tool calls throw `AUTH_CONFIG_ERROR` at first use. LOINC, RxNorm, MeSH have no auth.
- **SNOMED is feature-flagged off by default.** `src/utils/feature-flags.ts` gates the SNOMED tools and the SNOMED branch of crosswalk behind `ENABLE_SNOMED_TOOLS=true`. The historical public IHTSDO Snowstorm endpoint (`browser.ihtsdotools.org/snowstorm/snomed-ct`) was retired and now returns HTTP 410, so operators must also set `SNOMED_BASE_URL` to a working self-hosted Snowstorm. Optional `SNOMED_LANGUAGE` is passed through as the `Accept-Language` header.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SidneyBissoli/medical-terminologies-mcp](https://github.com/SidneyBissoli/medical-terminologies-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
