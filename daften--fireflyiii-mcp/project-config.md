---
trigger: always_on
description: **MCP server for Firefly III** is a TypeScript implementation of an MCP (Model Context Protocol) server that bridges Claude Code to a running Firefly III personal finance instance. This is a greenfield, open-source project (MIT license).
---

# MCP server for Firefly III — Project Documentation

## Project Overview

**MCP server for Firefly III** is a TypeScript implementation of an MCP (Model Context Protocol) server that bridges Claude Code to a running Firefly III personal finance instance. This is a greenfield, open-source project (MIT license).

Users can query their finances in natural language through Claude, getting answers about accounts, transactions, budgets, categories, bills, piggy banks, and financial insights without writing queries themselves.

**Current state:** 140 tools across 14 groups, full CRUD, stdio and HTTP (OAuth or PAT) transports, tool filtering via `--preset`/`--groups`/`--read-only`.

### Architecture at a glance

```
MCP client (Claude Code / Desktop / ...)
        │
        │  stdio                          HTTP (StreamableHTTP, stateless)
        ▼                                 ▼
  StdioServerTransport          OAuth proxy (src/http.ts)
        │                         · /.well-known metadata, /oauth/{authorize,token,register,callback}
        │                           (404 instead, if FIREFLY_OAUTH_CLIENT_ID is unset — PAT-only mode)
        │                         · substitutes redirect_uri, Bearer guard
        │                         · per-request token via AsyncLocalStorage
        └────────────┬────────────┘
                     ▼
            McpServer (src/server.ts)
                     │  registerAllTools (src/tools/index.ts)
                     │  · TOOL_GROUPS / PRESETS filtering, read-only proxy
                     ▼
      Tool groups (src/tools/*.ts, 14 groups / 140 tools)
        · defineTool wrapper: zod validation, error formatting (src/tools/_helpers.ts)
        · autocomplete prompts with per-user TTL cache
                     │
                     ▼
        FireflyClient (src/client.ts)
        · Bearer auth, 30s timeout, FireflyError → friendly messages
                     │
                     ▼
        Transform layer (src/transform.ts)
        · unwraps JSON:API envelopes → flat objects + pagination
                     │
                     ▼
            Firefly III REST API (/api/v1)
```

---

## Tech Stack

- **Language:** TypeScript (ESM modules, strict mode)
- **Runtime:** Node.js 20+ with tsx for development
- **MCP SDK:** `@modelcontextprotocol/sdk` v1.29.0+
- **Validation:** Zod for input schemas (inline in each tool file)
- **Testing:** Vitest for unit and integration tests
- **Build:** TypeScript compiler to ES2022 with source maps
- **Transport:** stdio (default) or HTTP (`--transport http`); HTTP is stateless StreamableHTTP with OAuth proxy

---

## Environment Variables

**stdio transport:**
```
FIREFLY_URL       String, required. Base URL of Firefly III instance (no trailing slash).
FIREFLY_TOKEN     String, required. Personal Access Token from Firefly III Options → Remote access and tokens → Create new token.
```

**HTTP transport:**
```
FIREFLY_URL                String, required. Base URL of Firefly III instance (no trailing slash).
FIREFLY_OAUTH_CLIENT_ID    String, optional. OAuth client ID from Firefly III Options → Remote access and tokens → Create New Client.
                           Omit to run in PAT-only mode (no OAuth proxy surface; see below).
MCP_BASE_URL               String, required when not listening on loopback AND FIREFLY_OAUTH_CLIENT_ID is set. Public base URL of this server.
```

In HTTP mode, the Bearer token is always resolved per-request from the Authorization header — either set by the MCP client after completing the OAuth flow, or supplied directly as a Firefly III Personal Access Token when `FIREFLY_OAUTH_CLIENT_ID` is unset (PAT-only mode). `FIREFLY_TOKEN` is not used in HTTP mode.

**Optional (both transports):**
```
FIREFLY_DEBUG     Set to "true" or "1" to emit verbose autocomplete tracing to stderr. Off by default.
```

**Tool-filtering fallbacks (both transports)** for the `--preset`/`--groups`/`--read-only` flags. The CLI flag always wins when both are set.
```
MCP_PRESET      String. Named preset; same values as --preset. Mutually exclusive with MCP_GROUPS.
MCP_GROUPS      String. Comma-separated group names; same values as --groups. Empty/whitespace-only is treated as unset.
MCP_READ_ONLY   "true" or "1" (case-insensitive, trimmed) enables read-only mode; any other value is ignored.
```

Store credentials in `.env` file (which is gitignored). The `.env.example` template shows what's needed.

---

## CLI Flags

Parsed by `src/args.ts` and passed to `createServer` as `filterOptions`.

| Flag | Default | Description |
|------|---------|-------------|
| `--transport stdio|http` | `stdio` | Transport mode |
| `--host <host>` | `127.0.0.1` | Bind address (HTTP only) |
| `--port <n>` | `3000` | Listen port (HTTP only; auto-increments on EADDRINUSE) |
| `--preset <name>` | — | Load a named tool subset (see Filtering) |
| `--groups <list>` | — | Comma-separated group names; cannot combine with `--preset` |
| `--read-only` | false | Filter any selection to read-only tools (`get_*`, `search_*`, `test_*`) |

---

## File Structure

```
fireflyiii-mcp/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daften/fireflyiii-mcp](https://github.com/daften/fireflyiii-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
