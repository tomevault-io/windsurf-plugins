---
trigger: always_on
description: This file is for **AI coding agents** (Cursor, Claude Code, Copilot, etc.) wiring RankMySEO into an application.
---

# AGENTS.md — RankMySEO integrator guide

This file is for **AI coding agents** (Cursor, Claude Code, Copilot, etc.) wiring RankMySEO into an application.

## Package decision tree

| Goal | Install |
| --- | --- |
| API-only backend (Hono) | `@rankmyseo/core`, `@rankmyseo/storage`, `@rankmyseo/server-hono`, `hono` |
| API-only backend (Express) | `@rankmyseo/core`, `@rankmyseo/storage`, `@rankmyseo/server-express`, `express` |
| API-only backend (Next App Router) | `@rankmyseo/core`, `@rankmyseo/storage`, `@rankmyseo/server-next` |
| API-only backend (Nitro / Nuxt) | `@rankmyseo/core`, `@rankmyseo/storage`, `@rankmyseo/server-nitro`, `h3` |
| SvelteKit / Astro (native Request/Response) | `@rankmyseo/core`, `@rankmyseo/storage`, `@rankmyseo/server` — see `examples/` |
| Postgres via Prisma | add `@rankmyseo/storage-prisma` (+ `@prisma/client`, `prisma`) instead of / beside Drizzle |
| Postgres via Kysely | add `@rankmyseo/storage-kysely` |
| + Headless React hooks | add `@rankmyseo/react`, `react` |
| + Headless Vue 3 composables | add `@rankmyseo/vue`, `vue` |
| + Headless Svelte stores | add `@rankmyseo/svelte`, `svelte` |
| + Framework-neutral HTTP client | add `@rankmyseo/client` (Astro/vanilla / custom adapters) |
| + Browser page collector | add `@rankmyseo/collector` |
| + Prebuilt dashboard widgets | add `@rankmyseo/ui`, `react-dom` |
| + CLI (`init`, `migrate`, `schedule`, `doctor`) | add `@rankmyseo/cli` (dev) or use `rankmyseo` meta installer |
| + AI chat tools / MCP | add `@rankmyseo/agent`, configure `agentModel` on the server handler |
| + SEO regression CI gate | add `@rankmyseo/cli` (+ `@rankmyseo/scanner`), configure `regression` in config |
| Full stack shortcut | `npx rankmyseo install --yes --preset recommended` |

The **recommended** preset installs: `@rankmyseo/core`, `@rankmyseo/storage`, `@rankmyseo/server-hono`, `@rankmyseo/react`, `@rankmyseo/cli` (+ peers `hono`, `react`).

## Support matrix

| Surface | Status |
| --- | --- |
| Node.js ≥ 20 full stack | Yes |
| Edge / Cloudflare Workers (full stack) | **No** |
| Adapters: Hono, Express, Next, Nitro | Yes |
| SvelteKit / Astro via `@rankmyseo/server` `createHandler` | Yes (`examples/`) |
| React UI widgets (`@rankmyseo/ui`) | Yes |
| Vue / Svelte headless | Yes |
| Vue / Svelte UI widgets | Deferred |
| Storage: sqlite / postgres / prisma / kysely | Yes |
| MySQL | No |
| SEO regression CLI | Yes — see [SEO Regression wiki](https://github.com/madebyaris/rankmyseo/wiki/SEO-Regression) |

Docs reference app (dogfoods `@rankmyseo/client`): `pnpm --filter @rankmyseo/docs dev`.

## CLI commands (real binary names)

| User-facing | Direct CLI binary |
| --- | --- |
| `npx rankmyseo init` | `npx rankmyseo-cli init` |
| `npx rankmyseo migrate` | `npx rankmyseo-cli migrate` |
| `npx rankmyseo schedule` | `npx rankmyseo-cli schedule` |
| `npx rankmyseo doctor` | `npx rankmyseo-cli doctor` |
| `npx rankmyseo regression check` | `npx rankmyseo-cli regression check` |

Do **not** use `npx @rankmyseo/cli` — that is not a valid binary name.

Global flags: `--json` (machine-readable output), `--version`.

## Environment variables

| Variable | Used by | Purpose |
| --- | --- | --- |
| `OPENAI_API_KEY` | Your server bootstrap | LLM for `POST /agent/chat` (bring your own provider wiring) |
| `DATABASE_URL` / `RANKMYSEO_DATABASE_URL` | `rankmyseo-mcp` bin | SQLite/Postgres URL for MCP stdio server |
| `TENANT_ID` / `RANKMYSEO_TENANT_ID` | MCP bin | Default tenant scope |
| `PROJECT_ID` / `RANKMYSEO_PROJECT_ID` | MCP bin | Default project scope |
| `RANKMYSEO_MCP_ALLOW_MUTATIONS` | MCP bin | Set to `1` to register mutating MCP tools (read-only by default) |

RankMySEO does not read OpenAI keys internally — you pass a `LanguageModel` to `createHandler` / `createRankMySeoApp`.

## HTTP scope headers

Most API routes require:

```
x-tenant-id: <tenant>
x-project-id: <project>
```

**These headers select scope — they do not authenticate.** Pass `authorize(request, scope)` to `createHandler` / `createRankMySeoApp` to validate the caller. Return a `Response` to deny.

Mount under a subpath with `basePath: "/api/rankmyseo"` on `createHandler` / `createRankMySeoApp` (Node.js ≥ 20; full stack is not edge/Workers compatible).

**Exempt routes** (no scope headers required; default config tenant/project is used):

- `GET /sitemap.xml`
- `GET /llms.txt`

**Special case:**

- `GET /` — scope headers optional; uses config defaults unless headers are provided

Disabled features:

- `POST /collect`, `/blog/*` → **403** when feature off
- `GET /sitemap.xml`, `/llms.txt` → **404** when feature off

## Minimal Hono + agent snippet

```ts
import { defineConfig } from "@rankmyseo/core";
import { createStore } from "@rankmyseo/storage";
import { createRankMySeoApp } from "@rankmyseo/server-hono";
import { openai } from "@ai-sdk/openai";

const store = createStore("sqlite://./data/rankmyseo.sqlite");

await store.projects.create({
  id: "project-1",
  tenantId: "tenant-a",
  name: "My Site",
  domain: "example.com",
});

const config = defineConfig({
  databaseUrl: "sqlite://./data/rankmyseo.sqlite",
  tenantId: "tenant-a",
  projectId: "project-1",
  dataSources: [{ provider: "fixture", default: true }],

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/rankmyseo](https://github.com/madebyaris/rankmyseo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
