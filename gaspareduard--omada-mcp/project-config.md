---
trigger: always_on
description: > This file is the single source of truth for AI agents working on this repository.
---

# TP-Link Omada MCP Server — AI Agent Instructions

> This file is the single source of truth for AI agents working on this repository.
> For human contributors, see [CONTRIBUTING.md](./CONTRIBUTING.md) and [CODEBASE.md](./CODEBASE.md).

## Repository Purpose

Security-focused MCP server exposing TP-Link Omada controller APIs. Written in TypeScript/Node.js. Production baseline is `stdio`; HTTP transport is in the codebase only as an explicitly unsafe, lab-only path.

## Tooling and Runtime

- Node.js 24.x — CI target and minimum runtime.
- TypeScript 5.9 — `module`/`moduleResolution`: `NodeNext`.
- Zod 3.x — config validation (MCP SDK requires Zod 3 APIs).
- Biome 2.x — linting and formatting (not ESLint/Prettier).
- Vitest 4.x — test framework.

## Environment Variables

Reference `.env.example` for the full list. All vars are loaded and validated in `src/config.ts`.

### Required

- `OMADA_BASE_URL` — controller URL (e.g. `https://omada.local`)
- `OMADA_CLIENT_ID` — OAuth2 client ID
- `OMADA_CLIENT_SECRET` — OAuth2 client secret
- `OMADA_OMADAC_ID` — controller ID

### Optional — Omada

- `OMADA_SITE_ID` — default site ID
- `OMADA_STRICT_SSL` (default: `true`) — enforce SSL certificate validation
- `OMADA_TIMEOUT` (default: `30000`) — request timeout ms
- `OMADA_CAPABILITY_PROFILE` (default: `safe-read`) — `safe-read` / `ops-write` / `admin` / `compatibility`
- `OMADA_TOOL_CATEGORIES` — explicit category override, e.g. `clients:rw,devices-all:r`

### Optional — Server

- `MCP_SERVER_LOG_LEVEL` (default: `info`) — `debug` / `info` / `warn` / `error`
- `MCP_SERVER_LOG_FORMAT` (default: `plain`) — `plain` / `json` / `gcp-json`
- `MCP_SERVER_USE_HTTP` (default: `false`) — lab-only HTTP mode
- `MCP_UNSAFE_ENABLE_HTTP` (default: `false`) — required acknowledgement for HTTP mode

### HTTP mode only (when both HTTP flags are `true`)

- `MCP_HTTP_PORT` (default: `3000`)
- `MCP_HTTP_BIND_ADDR` (default: `127.0.0.1`) — loopback only in safe baseline
- `MCP_HTTP_PATH` (default: `/mcp`)
- `MCP_HTTP_ENABLE_HEALTHCHECK` (default: `true`)
- `MCP_HTTP_HEALTHCHECK_PATH` (default: `/healthz`)
- `MCP_HTTP_ALLOW_CORS` (default: `true`)
- `MCP_HTTP_ALLOWED_ORIGINS` (default: `127.0.0.1, localhost`)
- `MCP_HTTP_NGROK_ENABLED` (default: `false`) — disabled in safe baseline

## Code Structure

```
src/
  index.ts           Startup — picks stdio or HTTP transport
  config.ts          All env var loading + Zod validation (single source of truth)
  env.ts             Raw process.env — only used by config.ts
  types.ts           Top-level shared types
  omadaClient/       Omada HTTP API layer (organized by API tag)
    index.ts         OmadaClient class
    auth.ts          OAuth2 client credentials + token caching
    request.ts       Axios base wrapper
    *.ts             Operations classes per API tag
  server/
    common.ts        Shared tool/prompt registration logic
    stdio.ts         stdio transport (production)
    http.ts          HTTP coordinator (lab-only)
    stream.ts        Streamable HTTP (MCP 2025-03-26, lab-only)
  tools/
    index.ts         Tool registration (imports all tools)
    types.ts         Shared tool-layer types
    *.ts             One file per MCP tool
  utils/
    logger.ts        Pino wrapper — use this, never console.log
    config-validations.ts  Zod validators for config values
    pagination-schema.ts   Reusable Zod schema for list args
docs/openapi/        Reference OpenAPI specs (READ ONLY)
tests/               Mirrors src/ 1:1 (enforced by CI)
OMADA_TOOLS.md       Ground-truth backlog of all 1648 API operations
CODEBASE.md          Full architecture + "how to add a tool" walkthrough
```

## Mandatory Coding Rules

### Environment Variables
- **Never** call `process.env.` outside `src/env.ts` and `src/config.ts`.
- All env vars must go through `loadConfigFromEnv()` in `src/config.ts`.

### TypeScript
- No `any` — use `unknown` and narrow with type guards or Zod.
- All imports use `.js` extension (NodeNext ESM requirement).
- Explicit return types on public functions.

### Logging
- Always use `src/utils/logger.ts`. Never `console.log`.

### Pagination
- Always reuse `src/utils/pagination-schema.ts` for list tool args.

### Formatting
- Biome handles formatting + linting. LF line endings required.
- Run `npm run format` to fix CRLF automatically.

## API Validation — MANDATORY Before Implementing Any Tool

1. **Verify the endpoint exists** in `docs/openapi/<tag>.json` before writing any code.
2. **Use `OMADA_TOOLS.md` as ground truth** — every tool there has a verified route.
3. **If not in `OMADA_TOOLS.md` and not in `docs/openapi/`** — do NOT implement it.
4. **Never infer or guess API routes** — only implement against verified spec paths.
5. **Never edit `docs/openapi/*.json`** — they are reference-only.
6. **Use individual tag files** (e.g. `03-device.json`), not `00-all.json` (too large).

## Adding a New Tool

1. Verify endpoint in `docs/openapi/<tag>.json` or `OMADA_TOOLS.md`.
2. Add method to the appropriate `src/omadaClient/<area>.ts` Operations class.
3. Expose it on `OmadaClient` in `src/omadaClient/index.ts`.
4. Create `src/tools/<toolName>.ts` following existing tool pattern.
5. Register it in `src/tools/index.ts` under the correct category block.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gaspareduard/Omada-mcp](https://github.com/gaspareduard/Omada-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
