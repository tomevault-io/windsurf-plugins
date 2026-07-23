---
trigger: always_on
description: A monorepo template for building full-stack JavaScript applications with Node.js, Express, and Vue. It combines backend and frontend apps in `apps/`, and shared reusable code in `common/`. Designed as an updateable template — custom code is isolated so upstream template updates can be merged cleanly.
---

# CLAUDE.md

## Project overview

A monorepo template for building full-stack JavaScript applications with Node.js, Express, and Vue. It combines backend and frontend apps in `apps/`, and shared reusable code in `common/`. Designed as an updateable template — custom code is isolated so upstream template updates can be merged cleanly.

- Node.js 24+ required, npm 11+ required
- Fully ES Modules — no CommonJS
- In `.ts` files: use TypeScript type annotations — do **not** use JSDoc for types
  - used mainly in `apps/*` (with ts), `common/compiled/*` and`scripts/*`
- In `.js` files: use JSDoc (`/** @type */`, `/** @param */`, etc.) for type hints and IDE autocomplete
  - used mainly in `apps/*` (with js) and `common/vanilla/*`
- TypeScript runs natively via Node 24 (`node file.ts`) — no build step needed for scripts

## Repository structure

```
apps/                # backend and frontend apps (npm workspace)
  sample-api/        # sample backend app — copy and rename, do not develop here directly
  base-iam/          # sample IAM service — auth, RBAC/FGA, user management (SAML/OIDC)
  cron/              # HTTP-triggered cron microservice — plain Express app (its own preRoute/postRoute), no internal scheduler; an external scheduler hits routes like `POST /cron/process-outbox`. Auth is a separate bearer-token scheme (`CRON_API_KEY`), not the main JWT/RBAC/FGA system
  sample-mcp/        # MCP server example
  sample-a2a/        # agent-to-agent sample — supervisor (Claude) + specialist (OpenAI RAG) over MCP
  sample-rag/        # RAG sample — document ingestion + retrieval via MCP tools + OpenAI generation
  sample-vue-full/   # full-featured sample Vue app (port 8081)
  sample-vue-minimal/ # minimal Vue app (port 8080)
  sample-common/     # internal shared backend code for apps/* workspaces (@apps/sample-common)
common/              # shared reusable code (npm workspaces)
  compiled/          # modules that require a build step
    node/            # Node.js modules, Express middleware and services (@common/node)
    vue/             # Vue-specific shared modules (@common/vue)
  vanilla/           # plain JS modules, no build step
    iso/             # isomorphic utilities, runs in Node and browser (@common/iso)
    web/             # browser-only utilities and web components
  schemas/           # shared zod schemas — not an npm workspace (imported directly), but has its own package.json with docs:generate/validate/make-html scripts (run from within common/schemas/)
db/                  # database schemas, migrations, seeds — separate npm workspaces, one per schema
  dev.db/            # PGlite data file for local dev (gitignored) — served by scripts/db-mocks
  sample/            # @db/sample — public schema: schema.ts, drizzle.config.ts, migrations, seeds (consumed by sample-api, cron)
  iam/               # @db/iam — iam schema: schema.ts, drizzle.config.ts, migrations, seeds (consumed by base-iam)
  audit/             # @db/audit — audit schema: schema.ts, drizzle.config.ts, migrations (SOC2/HIPAA trail)
docs/                # project documentation
generated/           # gitignored scratch folder for local build/runtime artifacts — only .gitignore/README.md are tracked
scripts/             # code/OpenAPI generation tooling, service mocks (npm workspace)
  generators/        # @tools/generators — generate-crud.ts (Drizzle schema → Zod/routes/controllers) and generate-openapi.ts
  service-mocks/     # redis/kafka/SAML/OIDC mocks
  db-mocks/          # serve-db.ts — local PGlite socket server backing db/sample, db/iam, db/audit
.github/             # GitHub Actions workflows and CONTRIBUTING.md
.githooks/           # native git hooks (pre-commit, pre-push)
```

`sample-mcp`, `sample-a2a`, and `sample-rag` are demo scripts (`npm run demo`, etc.), not long-running services — most have no real `test` script (stubbed to exit 0). `sample-mcp` is the MCP *server* (StreamableHTTP transport); `sample-a2a` and `sample-rag` are MCP *clients*, each with their own `mcp-client.ts` (duplicated, not shared). In `sample-a2a`, `supervisor.ts` and `specialist.ts` are two separate A2A protocol servers (ports 3100/3101, `/.well-known/agent.json` + `POST /` task endpoints) — the supervisor classifies and delegates to the specialist, which does the actual MCP-backed RAG query.

## Setup

```bash
# 1. install all workspace dependencies
npm i

# 2. configure git hooks (also runs automatically via npm prepare)
chmod +x .githooks/setup.sh && ./.githooks/setup.sh
# or manually:
# git config core.hooksPath .githooks

# 3. run the sample backend
cd apps/sample-api && npm run start

# 4. run the minimal Vue frontend
cd apps/sample-vue-minimal && npm run dev
```

## Common commands

```bash
# linting and formatting (biome)
npm run check          # biome check, no writes — safe to run in CI
npm run check:write    # biome check --write (auto-fix lint + format)
npm run ci             # biome ci (used in CI/CD)

# testing
npm run test:workspaces     # run tests in all workspaces


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ais-one/novex-kit](https://github.com/ais-one/novex-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
