---
trigger: always_on
description: This file provides guidance to Claude Code when working with the xpoz-ts repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with the xpoz-ts repository.

## Project Overview

TypeScript SDK for the Xpoz social media intelligence platform — wraps the xpoz-mcp server (MCP protocol) into a simple, typed async API.

## Repository Structure

```
xpoz-ts/
├── package.json            # Package config (tsup build, vitest tests)
├── tsconfig.json           # TypeScript config (ES2022, NodeNext modules, strict)
├── vitest.config.ts        # Test config (singleFork, 660s timeout)
├── README.md               # Documentation
└── src/
    ├── index.ts            # Public exports
    ├── version.ts          # VERSION string
    ├── client.ts           # XpozClient
    ├── pagination.ts       # PaginatedResult<T>
    ├── errors.ts           # Error hierarchy
    ├── version.ts          # VERSION string
    ├── cursor.ts           # CursorResult<T> for the live (cursor-paged) routes
    ├── rest/               # REST layer for the live routes (xpoz-api, not MCP)
    │   └── transport.ts    # fetch-based client + HTTP error mapping
    ├── mcp/                # MCP protocol layer
    │   ├── index.ts
    │   ├── transport.ts    # MCP Streamable HTTP transport wrapper
    │   └── polling.ts      # Operation polling (5s interval, configurable timeout)
    ├── transform/          # Response data processing
    │   ├── index.ts
    │   └── responseParser.ts # Response parsing helpers
    ├── config/             # Configuration values
    │   ├── index.ts
    │   ├── constants.ts    # DEFAULT_SERVER_URL, ENV vars, timeouts
    │   └── tools.ts        # MCP tool name constants
    ├── types/              # TypeScript interfaces
    │   ├── index.ts
    │   ├── common.ts       # PaginationInfo
    │   ├── twitter.ts      # TwitterPost, TwitterUser
    │   ├── instagram.ts    # InstagramPost, InstagramUser, InstagramComment
    │   └── reddit.ts       # RedditPost, RedditUser, RedditComment, RedditSubreddit, composites
    └── namespaces/         # Platform method groups
        ├── base.ts         # BaseNamespace (shared logic)
        ├── twitter.ts      # TwitterNamespace (12 methods)
        ├── instagram.ts    # InstagramNamespace (9 methods)
        └── reddit.ts       # RedditNamespace (9 methods)
```

## Development Commands

All commands run from the repo root. Requires Node.js 18+.

```bash
# Install dependencies
npm install

# Build (ESM + CJS + .d.ts)
npm run build

# Type check
npm run typecheck

# Verify imports
node -e "import('xpoz').then(m => console.log(Object.keys(m)))"
```

## Testing

Tests hit the live Xpoz API and require a valid API key:

```bash
XPOZ_API_KEY=your-api-key npm test
```

Tests must run in a **single sequential process** — `vitest.config.ts` enforces `singleFork: true`. The Xpoz API rate-limits concurrent connections, causing operations to queue beyond the timeout and cascade failures through the shared client.

The vitest timeout (660s) is intentionally higher than the client polling timeout (300s default) so the SDK raises a clean `OperationTimeoutError` instead of vitest killing the process.

## Architecture

### Transport Layer (`transport.ts`)

- `McpTransport`: Wraps MCP SDK's Streamable HTTP transport. Injects the API key as a Bearer token. Exposes `callTool(name, args) -> Promise<Record<string, unknown>>`.
- `connect()` must be called before any tool calls; `close()` tears down the session.

### Operation Polling (`polling.ts`)

Many xpoz-mcp tools return an `operationId` instead of immediate results (long-running queries). The poller:
- Calls `checkOperationStatus` every 5 seconds
- Returns result dict on `completed` status (or if `results`/`downloadUrl` present)
- Raises `OperationFailedError` / `OperationCancelledError` / `OperationTimeoutError`
- Default timeout: 300,000ms (configurable via `new XpozClient({ timeoutMs: N })`)

### Pagination (`pagination.ts`)

- `PaginatedResult<T>` wraps first-page data + `PaginationInfo` + a stored `fetchPage` callback
- `nextPage()`, `getPage(n)` fetch subsequent pages using `tableName` from the first response
- `exportCsv()` polls the export operation and returns a download URL

### Namespace Pattern

Each platform (Twitter, Instagram, Reddit) has a namespace class:
- Inherits `BaseNamespace` which provides `callAndMaybePoll()`, `buildArgs()`, `buildPaginatedResult()`
- `callAndMaybePoll()` handles `operationId` polling transparently
- `buildArgs()` filters out `undefined`/`null` values before sending to MCP

### Client Class (`client.ts`)

- `XpozClient`: Constructor accepts `{ apiKey?, serverUrl?, timeoutMs? }`. Reads `XPOZ_API_KEY` / `XPOZ_SERVER_URL` env vars as fallback.
- Must call `await client.connect()` before use.
- Supports `Symbol.asyncDispose` (`await using client = new XpozClient(...)`).
- Attaches `client.twitter`, `client.instagram`, `client.reddit` namespaces.

## Type Models

All types are plain TypeScript interfaces. All fields are optional (the API returns only requested fields). Fields are camelCase matching the MCP response format — no transformation is applied.

## Relationship to xpoz-mcp

This SDK is a client for the xpoz-mcp server. Key correspondences:

| SDK Concept | xpoz-mcp Source |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XPOZpublic/xpoz-ts-sdk](https://github.com/XPOZpublic/xpoz-ts-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
