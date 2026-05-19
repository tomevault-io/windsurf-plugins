---
trigger: always_on
description: This is the **Local Falcon MCP Server** (`@local-falcon/mcp`), a Model Context Protocol server that wraps the [Local Falcon API](https://docs.localfalcon.com). It enables AI agents to run geo-grid rank tracking scans, retrieve reports, manage campaigns, monitor Google Business Profiles, and analyze competitive positioning across Google Maps, Apple Maps, and AI search platforms.
---

# CLAUDE.md — @local-falcon/mcp

## Project Overview

This is the **Local Falcon MCP Server** (`@local-falcon/mcp`), a Model Context Protocol server that wraps the [Local Falcon API](https://docs.localfalcon.com). It enables AI agents to run geo-grid rank tracking scans, retrieve reports, manage campaigns, monitor Google Business Profiles, and analyze competitive positioning across Google Maps, Apple Maps, and AI search platforms.

**Package:** [`@local-falcon/mcp`](https://www.npmjs.com/package/@local-falcon/mcp) (npm)
**Version:** 1.4.3
**License:** MIT
**Runtime:** Node.js 18+
**Language:** TypeScript (strict mode)

## Architecture

```
index.ts          → Entry point. Transport selection (STDIO, SSE, HTTP), session management, OAuth 2.1
server.ts         → MCP tool registrations. Exports getServer() which creates McpServer with 37 tools
localfalcon.ts    → API client layer. All fetch functions, rate limiting, retry logic, timeout handling
oauth/            → OAuth 2.1 authorization server (routes, provider, config, state/client stores)
```

### Key Design Patterns

- **`server.ts`** exports a single `getServer(sessionMapping)` function that creates and returns an `McpServer` instance with all 37 tools registered via `server.tool(name, description, zodSchema, annotations, handler)`. Every tool includes MCP tool annotations (`readOnlyHint`, `destructiveHint`) that signal to AI clients whether a tool reads data or modifies state.
- **`localfalcon.ts`** contains one exported function per API endpoint. Two call patterns:
  - **URL params (v1):** `new URL(endpoint)` → `url.searchParams.set()` → POST with JSON headers
  - **FormData (v2):** `new FormData()` → `form.append()` → POST with form body
- **API key resolution:** `getApiKey(ctx)` checks the session mapping first (for OAuth-authenticated remote sessions), then falls back to `process.env.LOCAL_FALCON_API_KEY` (for STDIO/local use).
- **`handleNullOrUndefined()`** converts null/undefined Zod outputs to empty strings before passing to API client functions. The API client functions then use `if (value)` guards to skip empty params.

### Infrastructure (localfalcon.ts)

| Component | Details |
|---|---|
| Rate Limiter | Sliding window, 5 requests per 1000ms |
| Retry | Exponential backoff, 3 retries, 1s initial delay. Retries on network errors, timeouts, 5xx responses |
| Timeout | 30s default (`DEFAULT_TIMEOUT_MS`), 60s for long operations (`LONG_OPERATION_TIMEOUT_MS`) |
| JSON Parsing | `safeParseJson()` helper with error logging |

### Transport Modes

Started via CLI argument to `index.ts`:

| Mode | Command | Description |
|---|---|---|
| `stdio` (default) | `npm run start` or `npm run start:stdio` | Standard I/O for local MCP clients |
| `sse` | `npm run start:sse` | Server-Sent Events, OAuth 2.1 protected |
| `http` | `npm run start:http` | Streamable HTTP, OAuth 2.1 protected |
| `HTTPAndSSE` | `npm run start:HTTPAndSSE` | Both HTTP and SSE on same server |

Remote modes (SSE, HTTP) use OAuth 2.1 with PKCE for authentication. The server implements RFC 8414 (Authorization Server Metadata), RFC 9728 (Protected Resource Metadata), and RFC 7591 (Dynamic Client Registration).

## Tool Inventory

### Reports — List & Retrieve (20 tools, all support `fieldmask`)

| Tool | Description |
|---|---|
| `listLocalFalconScanReports` | List scan reports. Filter by placeId, keyword, platform, gridSize, date range, campaignKey |
| `getLocalFalconReport` | Get a specific scan report by report_key |
| `listLocalFalconTrendReports` | List trend reports. Filter by placeId, keyword, platform, date range |
| `getLocalFalconTrendReport` | Get a specific trend report by report_key |
| `listLocalFalconLocationReports` | List location reports. Filter by placeId, keyword, date range |
| `getLocalFalconLocationReport` | Get a specific location report by report_key |
| `listLocalFalconKeywordReports` | List keyword reports. Filter by keyword, date range |
| `getLocalFalconKeywordReport` | Get a specific keyword report by report_key |
| `getLocalFalconCompetitorReports` | List competitor reports. Filter by placeId, keyword, gridSize, date range |
| `getLocalFalconCompetitorReport` | Get a specific competitor report by report_key |
| `listLocalFalconCampaignReports` | List campaign reports. Filter by placeId, date range, runDate |
| `getLocalFalconCampaignReport` | Get a specific campaign report by report_key, optional run date |
| `listLocalFalconGuardReports` | List Falcon Guard reports. Filter by status, date range |
| `getLocalFalconGuardReport` | Get a specific guard report by placeId, optional date range |
| `listLocalFalconReviewsAnalysisReports` | List reviews analysis reports. Filter by placeId, frequency, reviewsKey |
| `getLocalFalconReviewsAnalysisReport` | Get a specific reviews analysis report by report_key |
| `listAllLocalFalconLocations` | List all saved locations in the account. Filter by query |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [local-falcon/mcp](https://github.com/local-falcon/mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
