---
trigger: always_on
description: This project implements a Model Context Protocol (MCP) server that exposes TP-Link Omada controller APIs. The server is written in TypeScript/Node.js and communicates with MCP clients over stdio and HTTP (Streamable HTTP transport).
---

# TP-Link Omada MCP Server - Developer Instructions

## Repository Purpose

This project implements a Model Context Protocol (MCP) server that exposes TP-Link Omada controller APIs. The server is written in TypeScript/Node.js and communicates with MCP clients over stdio and HTTP (Streamable HTTP transport).

## Tooling and Runtime

- Node.js 22 LTS (devcontainer base image `mcr.microsoft.com/devcontainers/typescript-node:1-22-bookworm`).
- TypeScript 5.9 with `module`/`moduleResolution` set to `NodeNext`.
- Zod 3.x for configuration validation (the MCP SDK currently expects Zod 3 APIs).
- Biome 2.x for linting and formatting.

## Environment Variables

Reference `.env.example`. Primary variables:

### Omada Client Configuration:

- `OMADA_BASE_URL` (required) - base URL of the Omada controller (e.g., `https://omada-controller.local`).
- `OMADA_CLIENT_ID` (required) - client ID for OAuth2 access.
- `OMADA_CLIENT_SECRET` (required) - client secret for OAuth2 access.
- `OMADA_OMADAC_ID` (required) - Omada controller ID (omadacId) to target.
- `OMADA_SITE_ID` (optional) - site ID for the Omada controller.
- `OMADA_STRICT_SSL` (default: `true`) - whether to enforce strict SSL certificate validation.
- `OMADA_TIMEOUT` (default: `30000`) - request timeout in milliseconds.

### MCP Generic Server Configuration:

- `MCP_SERVER_LOG_LEVEL` (default: `info`) - logging verbosity (`debug`, `info`, `warn`, `error`, `silent`).
- `MCP_SERVER_LOG_FORMAT` (default: `plain`) - log output format (`plain`,`json`, or `gcp-json`).
  - `plain` - human-readable text format.
  - `json` - structured JSON format.
  - `gcp-json` - structured JSON format compatible with Google Cloud Logging.
- `MCP_SERVER_USE_HTTP` (default: `false`) - whether to start the HTTP server instead of stdio.

### MCP Server HTTP Configuration, if `MCP_SERVER_USE_HTTP` is `true`:

- `MCP_HTTP_PORT` (default: `3000`) - port for the HTTP server.
- `MCP_HTTP_BIND_ADDR` (default: `127.0.0.1`) - bind address for the HTTP server (IPv4 or IPv6). For security, defaults to localhost.
- `MCP_HTTP_PATH` (default: `/mcp`) - base path for MCP HTTP endpoints.
- `MCP_HTTP_ENABLE_HEALTHCHECK` (default: `true`) - enable a healthcheck endpoint at the path indicated on `MCP_HTTP_HEALTHCHECK_PATH`.
- `MCP_HTTP_HEALTHCHECK_PATH` (default: `/healthz`) - path for the healthcheck endpoint.
- `MCP_HTTP_ALLOW_CORS` (default: `true`) - enable CORS for the HTTP server.
- `MCP_HTTP_ALLOWED_ORIGINS` (default: `127.0.0.1, localhost`) - comma-separated list of allowed origins for DNS rebinding protection. Must contain valid hostnames, IPv4, IPv6 addresses, or `*` to allow all origins (development only).
- `MCP_HTTP_NGROK_ENABLED` (default: `false`) - whether to use ngrok to expose the HTTP server publicly.
- `MCP_HTTP_NGROK_AUTH_TOKEN` (optional) - ngrok auth token, required if `MCP_HTTP_NGROK_ENABLED` is `true`.

## Code Structure

- `src/index.ts` — MCP Server startup, including both stdio and HTTP server initialization. The type of server is selected based on environment variables.
- `src/config.ts` — Environment variable loading and validation via Zod.
- `src/utils/` — Utility functions (e.g., logger, error handling).
- `src/omadaClient/` — Omada API interaction layer, organized by API tag (e.g., `src/omadaClient/user.ts`, `src/omadaClient/device.ts`). The main client class is in `src/omadaClient/index.ts`.
- `src/server/` — Code for each implementation of the MCP server:
  - `src/server/stdio.ts` - stdio transport implementation
  - `src/server/http.ts` - HTTP server coordinator
  - `src/server/stream.ts` - Streamable HTTP transport implementation (MCP 2025-03-26)
  - `src/server/common.ts` - common server logic shared across transports
- `src/types/` - centralized type definitions (API, MCP, errors)
- `src/tools/` - individual tool files and registration.
- `src/prompts/` - individual prompt files and registration.
- `docs/openapi/` — Reference OpenAPI specifications for Omada endpoints, split per API tag.
- `tests/` — Unit and integration tests. **The test folder structure MUST mirror the src folder structure.** For example:
  - `tests/utils/config-validations.test.ts` tests `src/utils/config-validations.ts`
  - `tests/server/http.test.ts` would test `src/server/http.ts`

## Testing

### Unit Tests

- The project uses **Vitest** as the test framework.
- All test files should be placed in the `tests/` directory with the `.test.ts` extension.
- The test folder structure **must mirror** the `src/` folder structure with strict 1:1 file matching.
  - Example: `src/tools/getClientDetail.ts` → `tests/tools/getClientDetail.test.ts`
  - Every `src/tools/<name>.ts` (except `index.ts` and `types.ts`) must have a matching `tests/tools/<name>.test.ts`.
  - Every `src/omadaClient/<name>.ts` (except `index.ts`) must have a matching `tests/omadaClient/<name>.test.ts`.
  - CI enforces this via `scripts/check-tool-tests.mjs` on every PR.
- Run tests with `npm test` or `npm run test:watch` for watch mode.
- Test coverage can be generated with `npm run test:coverage`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MiguelTVMS/tplink-omada-mcp](https://github.com/MiguelTVMS/tplink-omada-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
