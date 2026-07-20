---
trigger: always_on
description: An MCP (Model Context Protocol) server that gives AI assistants (Cursor, Claude, etc.) direct access to SAP Notes and Knowledge Base articles. It authenticates with SAP's systems via browser automation (Playwright) and exposes two tools: **search** and **get** for SAP Notes.
---

# SAP Note Search MCP Server - Project Overview for Claude Sessions

## What This Project Is

An MCP (Model Context Protocol) server that gives AI assistants (Cursor, Claude, etc.) direct access to SAP Notes and Knowledge Base articles. It authenticates with SAP's systems via browser automation (Playwright) and exposes two tools: **search** and **get** for SAP Notes.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│  MCP Client (Cursor / Claude Desktop / LibreChat)           │
│  Communicates via JSON-RPC 2.0                              │
└───────────┬──────────────────────────────┬──────────────────┘
            │ stdio transport              │ HTTP transport
            ▼                              ▼
┌───────────────────────┐    ┌──────────────────────────────┐
│  src/mcp-server.ts    │    │  src/http-mcp-server.ts      │
│  StdioServerTransport │    │  Express + StreamableHTTP    │
│  (for Cursor / CLI)   │    │  (for Docker / LibreChat)    │
└───────────┬───────────┘    └──────────────┬───────────────┘
            │                               │
            └──────────┬────────────────────┘
                       ▼
         ┌──────────────────────────┐
         │  MCP Tool Handlers       │
         │  search         │
         │  fetch            │
         └────────────┬─────────────┘
                      │
        ┌─────────────┴──────────────┐
        ▼                            ▼
┌──────────────────┐    ┌────────────────────────┐
│  src/auth.ts     │    │  src/sap-notes-api.ts  │
│  SapAuthenticator│    │  SapNotesApiClient     │
│  Playwright +    │    │  Coveo Search API      │
│  Certificate/    │    │  Raw Notes API         │
│  Username auth   │    │  Fallback strategies   │
└──────────────────┘    └────────────────────────┘
        │                            │
        ▼                            ▼
┌──────────────────────────────────────────────┐
│  SAP Systems                                  │
│  • accounts.sap.com (IAS - authentication)    │
│  • me.sap.com (notes, search, Coveo token)    │
│  • Coveo search engine (search API)           │
│  • launchpad.support.sap.com (note URLs)      │
└──────────────────────────────────────────────┘
```

## Source Files

### Core Server Files

| File | Purpose | Key Classes/Exports |
|------|---------|-------------------|
| `src/mcp-server.ts` | **Main entry point** - stdio MCP server for Cursor/CLI | `SapNoteMcpServer` |
| `src/http-mcp-server.ts` | HTTP MCP server for Docker/LibreChat deployments | `HttpSapNoteMcpServer` |
| `src/auth.ts` | Authentication via Playwright browser automation | `SapAuthenticator` |
| `src/sap-notes-api.ts` | SAP Notes search (Coveo) and retrieval (raw API) | `SapNotesApiClient` |
| `src/types.ts` | TypeScript interfaces and JSON schemas | `ServerConfig`, `AuthState`, `SapNote`, etc. |
| `src/logger.ts` | Pino-based logging with MCP mode detection | `logger`, `authLogger`, `apiLogger` |
| `src/schemas/sap-notes.ts` | Enhanced Zod schemas with LLM-optimized descriptions | `NoteSearchInputSchema`, `NoteGetInputSchema`, etc. |

### Test Files

| File | Purpose |
|------|---------|
| `test/test-auth.js` | Tests authentication flow |
| `test/test-sap-api.js` | Tests Coveo search and note retrieval |
| `test/test-mcp-server.js` | Tests MCP protocol interaction |
| `test/test-docker-debug.js` | Docker environment debugging |

### Configuration

| File | Purpose |
|------|---------|
| `.env` / `.env.example` | Environment variables (PFX_PATH, PFX_PASSPHRASE, etc.) |
| `tsconfig.json` | TypeScript compilation (ES2022, ESNext modules, strict) |
| `token-cache.json` | Cached authentication cookies (auto-generated) |

## Authentication Flow

The server supports two authentication methods:

### 1. Certificate-Based (Original)
- Uses a `.pfx` client certificate for SAP IAS
- Playwright launches a browser, presents the certificate during TLS handshake
- SAP authenticates automatically, browser extracts session cookies
- Cookies cached to `token-cache.json` with configurable TTL

### 2. Username/Password (New)
- Uses SAP username + password for form-based login
- Playwright fills the login form on `accounts.sap.com`
- Supports MFA/2FA with configurable timeout for manual code entry
- Credentials passed via env vars (`SAP_USERNAME`, `SAP_PASSWORD`) or MCP client config
- Falls back to certificate auth if username/password not provided

### Auth Priority
1. Check `token-cache.json` for valid cached token
2. If `SAP_USERNAME` + `SAP_PASSWORD` provided → username/password auth
3. If `PFX_PATH` + `PFX_PASSPHRASE` provided → certificate auth
4. Error if neither configured

## MCP Tools

### `search`
- **Input:** `{ q: string, lang?: 'EN' | 'DE' }`
- **Flow:** Coveo search API → fallback to direct ID lookup → fallback to SAP internal search
- **Output:** Ranked list of matching SAP Notes with metadata

### `fetch`
- **Input:** `{ id: string, lang?: 'EN' | 'DE' }`
- **Flow:** Playwright raw notes API → fallback to HTTP raw notes API

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marianfoo/mcp-sap-notes](https://github.com/marianfoo/mcp-sap-notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
