---
trigger: always_on
description: MCP OAuth compliance simulator — walks the exact path a real client would (unauthenticated probe → 401 challenge → discovery → registration → authorization → token exchange → MCP calls) and reports spec compliance at each step. Use when investigating auth failures, testing server compliance, or onboarding new OAuth MCP servers. Triggers on "debug oauth", "test oauth flow", "oauth compliance", "oauth-debug", "why is auth failing".
---


# MCP OAuth Compliance Simulator

Simulates the exact path a real MCP client walks — unauthenticated probe, 401 challenge parsing, metadata discovery, client registration, authorization, token exchange, and authenticated MCP calls. Reports pass/fail/warn at each step with RFC references.

## Usage

```bash
bun run ~/.claude/skills/mcp-oauth-debug/scripts/oauth-debug.ts <mcp-url>
```

Options:
- `--no-pkce` — skip PKCE (for servers that don't support S256)
- `--scopes "scope1 scope2"` — override discovered scopes
- `--port 9877` — override callback port (default 9877)

## The 6 phases

| Phase | What it does | Spec |
|-------|-------------|------|
| **1. Unauthenticated probe** | Sends `initialize` without auth. Checks for 401 + `WWW-Authenticate: Bearer` with `resource_metadata` hint | MCP spec, RFC 6750 §3, RFC 9728 §3.1 |
| **2. Protected resource metadata** | Follows discovery chain: challenge hint → path-inserted `.well-known` → root `.well-known`. Validates `authorization_servers` array and `resource` field match | RFC 9728 §3.3 |
| **3. Authorization server metadata** | Tries OAuth2 path-inserted → OIDC path-inserted → OIDC path-suffixed/root. Distinguishes 4xx (try next) from 5xx (stop). Validates issuer consistency | RFC 8414 §3, OIDC Discovery §4.1 |
| **4. Dynamic client registration** | Registers with DCR endpoint. Validates `client_id` in response | RFC 7591 |
| **5. Authorization + token exchange** | Full auth code flow with PKCE + resource indicator. Decodes JWT claims, checks audience/issuer consistency | RFC 7636, RFC 8707, RFC 9068 |
| **6. Authenticated MCP calls** | Calls `initialize` and `tools/list` with the obtained token | MCP spec |

## Correlation IDs (find this run's traces fast)

Every run threads two correlation IDs through the flow so the resulting waterfall is recoverable from the server's tracing backend in **one query**:

| ID | Where it goes | Survives |
|---|---|---|
| `probe_id` (8 hex) | OAuth `state="probe-<id>-<rand>"` + DCR `client_name="mcp-oauth-debug:<id>"` | Browser-driven legs (`/authorize`, `/callback`, the 302 back to localhost). RFC 6749 §10.12 mandates `state` be echoed verbatim. |
| `client_traceId` (32 hex) | W3C `traceparent` header on every script-driven `fetch` | Script-driven legs: probe POST, metadata discovery, DCR, token exchange, authenticated MCP calls. Server tracing infra continues the trace under one `TraceId`. |

Together they cover the **full** waterfall — neither alone is sufficient because the browser hop and the script hops use different correlation channels.

The boot banner prints both at script start:

```
  probe_id:       7c3d9f12
  client_traceId: a4b3...e1f2
  traceparent:    00-a4b3...e1f2-9c8b...d3a4-01
```

After the compliance summary, the script prints a paste-ready ClickHouse query:

```sql
SELECT Timestamp, ServiceName, SpanName,
       Duration / 1e6 AS ms, StatusCode,
       SpanAttributes['http.url'] AS url
FROM otel.otel_traces
WHERE Timestamp > now() - INTERVAL 30 MINUTE
  AND (
    SpanAttributes['http.url'] LIKE '%probe-7c3d9f12%'
    OR TraceId = 'a4b3...e1f2'
  )
ORDER BY Timestamp ASC
```

Drop that into your CH MCP / dashboard / `clickhouse-client` and you have the full chain — apps/auth's `/authorize` and `/callback` handlers, any user-worker `/__smithery/auth` dispatch, upstream IdP token-exchange spans, and the script's MCP calls — interleaved by timestamp.

**Useful queries off the same IDs:**

```sql
-- Logs for this run (joinable on TraceId; see observability skill)
SELECT Timestamp, ServiceName, SeverityText, Body
FROM otel.otel_logs
WHERE TraceId = '<client_traceId>' OR lower(Body) LIKE '%probe-<probe_id>%'
ORDER BY Timestamp ASC

-- Per-leg latency (find the slow span)
SELECT ServiceName, SpanName, Duration / 1e6 AS ms
FROM otel.otel_traces
WHERE Timestamp > now() - INTERVAL 30 MINUTE
  AND (SpanAttributes['http.url'] LIKE '%probe-<probe_id>%' OR TraceId = '<client_traceId>')
ORDER BY ms DESC
LIMIT 20
```

## After the script runs

The script does NOT call `tools/call` — that could trigger side effects on a real server. After reviewing the compliance summary:

1. Look at the `tools/list` output to identify a **safe, read-only tool** (e.g., a getter, list, or search tool)
2. Use the agent to call that tool manually if you need to verify auth works end-to-end on operations

## Interpreting the compliance summary

The script prints a structured summary at the end:

```
══════════════════════════════════════════════════════════════
  COMPLIANCE SUMMARY
══════════════════════════════════════════════════════════════
  [+] Pass: 12  [x] Fail: 1  [!] Warn: 2  [-] Skip: 0
──────────────────────────────────────────────────────────────
  [x] WWW-Authenticate header: 401 without WWW-Authenticate header
     → RFC 6750 §3: MUST include WWW-Authenticate on 401
  [!] Challenge resource_metadata: No resource_metadata in challenge

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smithery-ai/mcp-oauth-debug](https://github.com/smithery-ai/mcp-oauth-debug) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
