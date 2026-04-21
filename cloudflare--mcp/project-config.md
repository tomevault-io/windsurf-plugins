---
trigger: always_on
description: `cloudflare-mcp` is a token-efficient Model Context Protocol (MCP) server that exposes the entire Cloudflare API (~2,500 endpoints) using Cloudflare's **Code Mode** pattern. Instead of registering thousands of MCP tools, it uses just two tools (`search` and `execute`) that let agents write JavaScript to query the OpenAPI spec and call APIs — fitting all 2,500 endpoints into ~1,000 tokens.
---

# AGENTS.md

## Project overview

`cloudflare-mcp` is a token-efficient Model Context Protocol (MCP) server that exposes the entire Cloudflare API (~2,500 endpoints) using Cloudflare's **Code Mode** pattern. Instead of registering thousands of MCP tools, it uses just two tools (`search` and `execute`) that let agents write JavaScript to query the OpenAPI spec and call APIs — fitting all 2,500 endpoints into ~1,000 tokens.

**Production URL:** `mcp.cloudflare.com`

## MCP specification compliance

When modifying MCP or OAuth functionality, **always check the latest published MCP specification**:

- **Specification:** https://modelcontextprotocol.io/specification/2025-11-25
- **Authorization section:** https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization

## Repository structure

```
cloudflare-mcp/
├── src/
│   ├── index.ts                   # Worker entry point & OAuth/Hono routing
│   ├── server.ts                  # MCP server setup & tool registration
│   ├── executor.ts                # Code executor (Worker Loader API)
│   ├── spec-processor.ts          # OpenAPI spec fetching & $ref resolution
│   ├── truncate.ts                # Response truncation (~6K token limit)
│   ├── auth/
│   │   ├── types.ts               # Auth props schemas (Zod discriminated union)
│   │   ├── api-token-mode.ts      # Direct Cloudflare API token support
│   │   ├── cloudflare-auth.ts     # PKCE & OAuth utilities
│   │   ├── oauth-handler.ts       # OAuth authorization flow
│   │   ├── scopes.ts              # OAuth scope definitions (120+ scopes)
│   │   └── workers-oauth-utils.ts # OAuth provider helpers
│   └── tests/
│       ├── index.test.ts
│       ├── auth/
│       ├── executor.test.ts
│       ├── spec-processor.test.ts
│       └── truncate.test.ts
├── site/                          # Marketing/docs website (Astro + React)
├── scripts/
│   └── seed-r2.ts                 # Seed OpenAPI spec to R2 bucket
├── .github/workflows/
│   ├── ci.yml                     # PR validation
│   └── bonk.yml                   # AI code review
├── wrangler.jsonc                 # Workers config (dev/staging/prod)
├── .oxfmtrc.json                  # oxfmt formatter config
└── README.md
```

## Setup

```bash
npm install    # Install dependencies
```

Node 22+ required.

## Commands

| Command                | What it does                                  |
| ---------------------- | --------------------------------------------- |
| `npm run dev`          | Start local dev server (wrangler dev)         |
| `npm run deploy`       | Deploy to staging                             |
| `npm run deploy:prod`  | Deploy to production                          |
| `npm run types`        | Generate worker type definitions              |
| `npm run typecheck`    | TypeScript type checking (no emit)            |
| `npm run lint`         | Lint with oxlint                              |
| `npm run format`       | Format with oxfmt                             |
| `npm run format:check` | Check formatting without modifying            |
| `npm run test`         | Run vitest test suite                         |
| `npm run test:watch`   | Run vitest in watch mode                      |
| `npm run check`        | Run all checks (format, lint, typecheck, test)|
| `npm run seed:staging` | Seed OpenAPI spec to staging R2               |
| `npm run seed:prod`    | Seed OpenAPI spec to production R2            |

## Code standards

### TypeScript

- Strict mode enabled
- Target: ES2022, Module: ESNext
- Runtime validation with Zod for auth props and external data

### Formatting & linting

- **oxfmt** for formatting: single quotes, no semicolons, no trailing commas
- **oxlint** for linting
- Run `npm run format` before committing

### Naming conventions

- `PascalCase` for classes, interfaces, types, enums
- `camelCase` for functions, methods, variables
- `SCREAMING_SNAKE_CASE` for constants

## Architecture

### Two-tool Code Mode pattern

The core innovation: instead of 2,500 MCP tools (~244K tokens), two tools handle everything:

1. **`search` tool** — Agents write JavaScript to query the pre-resolved OpenAPI spec (all `$ref`s inlined). Runs in an isolated worker with no network access.
2. **`execute` tool** — Agents write JavaScript using `cloudflare.request()` to call discovered endpoints. Runs in an isolated worker with outbound restricted to Cloudflare API URLs only.

### Worker Loader API

Code execution uses Cloudflare's Worker Loader API to dynamically create isolated worker instances. The API token is passed via props (never enters user code isolate). A `globalOutbound` service restricts network access.

### Authentication

Two modes via Zod discriminated union (`AuthProps`):

- **OAuth mode** (default): Uses `@cloudflare/workers-oauth-provider` with PKCE. Supports both account-scoped and user-scoped tokens.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflare/mcp](https://github.com/cloudflare/mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
