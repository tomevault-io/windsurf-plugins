---
trigger: always_on
description: Top-level rules and information for the project
---

# Deno MCP Template Development Guide

Deno version: ^2.6.0

## Build/Development Commands

Start: `deno task start`
Dev mode: `deno task dev`
Format, lint, and type-check code: `deno task ci`
Rebuild MCP App HTML (after editing `mcp-ui/`): `deno task build:mcp-ui` (Deno installs npm deps from `mcp-ui/deno.json` `imports` into `mcp-ui/node_modules` using `mcp-ui/deno.lock`; no Node.js)
Tests with coverage: `deno task test:coverage`
Benchmarks: `deno task bench`

## Architecture

### `src/mcp/` vs `src/app/`

- `src/mcp/` — MCP server: prompts, resources, tools, tasks, apps, server wiring.
- `src/app/` — Host shell: transport bootstrapping, HTTP (Hono on `Deno.serve`), lifecycle, permissions preflight, KV, cron, signals.

### HTTP stack

Middleware includes rate limiting, CORS, optional bearer auth (see `src/app/http/hono.ts`), security headers, request timeouts, and session handling.

### Transport-scoped `McpServer` instances

The MCP TypeScript SDK allows **one active transport per protocol instance**. This template creates **a separate `McpServer` per transport binding**, not one shared instance:

- **STDIO:** one long-lived `McpServer` for the lifetime of the STDIO transport (`createApp` in `src/app/app.ts`).
- **HTTP (streamable):** **one new `McpServer` per MCP HTTP session** (factory in `createHttpServer` / Hono setup).

Process-wide state — **Deno KV**, **resource subscription tracker**, **task queue worker**, etc. — lives **outside** those instances. The factory receives the same `McpServerFactoryContext` each time (including shared `subscriptions`) so STDIO and all HTTP sessions stay consistent.

### Security and transport toggles

- **Host header validation** (aligned with MCP SDK Hono middleware): on loopback listen addresses, `Host` must be `localhost`, `127.0.0.1`, or `[::1]` (port ignored). With `--dnsRebinding` and configured allowed hosts, that list applies instead. Binding to all interfaces without DNS rebinding allowlists logs a one-time startup warning.
- **DNS rebinding protection** on the streamable HTTP transport is opt-in: `MCP_DNS_REBINDING=true` or `--dnsRebinding` (with `--origin` / `--host` per CLI rules). Use `MCP_ALLOWED_ORIGINS` and `MCP_ALLOWED_HOSTS` for Origin and transport header checks.
- CORS allowed origins default to **empty**; browser clients need `MCP_ALLOWED_ORIGINS` or `--origin`.
- HTTP: on by default; disable with `MCP_NO_HTTP=true` or `--no-http`.
- STDIO: on by default; disable with `MCP_NO_STDIO=true` or `--no-stdio`.

### Deno KV in this template

KV is built into Deno (`Deno.Kv`): file-backed locally, managed service on [Deno Deploy](https://docs.deno.com/deploy/kv/manual/). Used here for HTTP event resumability, durable task state and results, delayed task queue, and the counter resource.

### Sandboxed `execute-code`

Runs untrusted TS/JS in [Deno Sandbox](https://docs.deno.com/sandbox/) (isolated VM, no net/fs/env). Set `DENO_DEPLOY_TOKEN` in `.env` (Deploy dashboard → Organization Tokens). Implementation: `src/mcp/tools/sandbox.ts`.

### Maintenance cron

`Deno.cron` jobs start from `src/app/app.ts`. Example: `cleanup-stale-tasks` every 15 minutes in `src/app/cron.ts` (marks stale working tasks failed). Requires `unstable: ["kv", "cron"]` in `deno.json`.

### Caveats

- `src/app/http/kvEventStore.ts` is a simple session-resumability helper, not a production-grade event store.
- Local tasks often use `deno run -A`; tighten permissions before production ([Deno security](https://docs.deno.com/runtime/fundamentals/security/)).
- With `--dnsRebinding`, configure origins/hosts via env/CLI or `src/shared/constants/http.ts`.
- Before shipping, review `static/.well-known/openapi.yaml`, `static/.well-known/llms.txt`, and `static/dxt-manifest.json`; set secrets in GitHub and Deno Deploy as needed.
- After `deno task setup`, do a final pass for template names and identifiers.

## Project Structure

The code is structured to be easily parsable by an AI agent. Files are grouped by feature, and ideally less than 200 lines of code.

```markdown
deno.json     # Project configuration
main.ts       # The main entry point
src/
├── app/
│   ├── http/
│   │   ├── handlers.ts             # HTTP handlers for the MCP server (GET, POST, etc.)
│   │   ├── hono.ts                 # Manages the Hono server, middleware, and routes
│   │   ├── hostHeaderMiddleware.ts # Host allowlist (localhost + explicit DNS rebinding hosts)
│   │   ├── httpBearerAuthMiddleware.ts # Optional bearer auth for HTTP routes
│   │   ├── kvEventStore.ts         # Simple Deno KV event store for session resumability
│   │   ├── mod.ts                  # The main entrypoint for the HTTP server
│   │   ├── urlElicitationRoutes.ts # Browser pages for URL-mode elicitation
│   │   └── transport.ts            # Manages StreamableHTTPServerTransports
│   ├── app.ts                      # The main application class
│   ├── cli.ts                      # Parses CLI args and env vars into an AppConfig object
│   ├── cron.ts                     # Scheduled jobs (e.g., stale task cleanup)
│   ├── permissions.ts              # Runtime permission preflight checks
│   ├── signals.ts                  # Signal handling for SIGINT, SIGTERM, etc.
│   └── stdio.ts                    # The STDIO transport & state manager
├── kv/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phughesmcr/deno-mcp-template](https://github.com/phughesmcr/deno-mcp-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
