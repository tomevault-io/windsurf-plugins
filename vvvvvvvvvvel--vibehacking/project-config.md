---
trigger: always_on
description: Use Caido MCP tools to inspect captured HTTP traffic, Sitemap, WebSocket messages, Replay, scopes, findings, filters, environments, and Tamper rules with compact projections, HTTPQL filters, regex excerpts, and safe evidence-focused workflows.
---


# Caido MCP

Use this skill when working with Caido through MCP tools.

## Operating Workflow

1. Start read-only: Sitemap or HTTP history discovery with small `fields`.
2. Narrow with Caido-native filters: HTTPQL for history, `filter` object for Sitemap.
3. Pull exact evidence by ID only after discovery.
4. Use `regex_excerpt` when only a token, ID, nonce, header, or error fragment is needed.
5. Send traffic only when the user expects active validation. Prefer `options.save=true` for later evidence.
6. Report IDs, filters, active sends, and remaining uncertainty.

Keep broad discovery body-light. Do not request raw bodies unless they are necessary evidence.

## HTTP History

Use `list_requests` for chronological traffic discovery. Its `filter` is a raw HTTPQL string.

Recent API traffic:

```json
{
    "limit": 50,
    "order": { "target": "req", "field": "id", "direction": "desc" },
    "filter": "req.path.cont:\"/api/\"",
    "fields": ["cursor", "id", "time", "request.method", "request.url", "response.status_code"]
}
```

Continue with `cursor: "<page_info.end_cursor>"`. For backward pagination use `direction: "before"` and `cursor: "<page_info.start_cursor>"`.

### HTTPQL Quick Examples

Use HTTPQL inside `list_requests.filter`, `summarize_request_auth_headers.filter`, and `summarize_request_cookies.filter`. If syntax is uncertain, call `get_httpql_help`.

```text
req.method.eq:"POST"
req.host.eq:"api.example.com" AND req.path.cont:"/api/"
resp.code.eq:401 OR resp.code.eq:403
req.raw.cont:"Authorization:" OR req.raw.cont:"X-API-Key:"
resp.raw.cont:"access_token" OR resp.raw.cont:"refresh_token"
req.path.cont:"/graphql" AND req.method.eq:"POST"
```

Prefer narrow host/path/method filters first, then add raw/header/body terms only when the candidate set is still too broad.

Exact evidence:

```json
{
    "ids": [123],
    "fields": ["id", "request.raw.text", "response.raw.text"],
    "serialization": {
        "include_body": true,
        "include_binary": false,
        "max_text_body_chars": 20000,
        "text_overflow_mode": "truncate"
    }
}
```

## Regex Excerpts

Use `serialization.regex_excerpt` plus `fields: ["match_context.excerpts"]` to return only matching fragments. If `request.body`, `response.body`, `request.raw`, or `response.raw` fields are also requested, Caido MCP ignores those body/raw fields and still returns excerpts.

Find likely API tokens without returning full responses:

```json
{
    "limit": 50,
    "filter": "req.host.eq:\"api.example.com\" AND (req.raw.cont:\"authorization\" OR req.raw.cont:\"x-api-key\" OR req.raw.cont:\"api_token\" OR resp.raw.cont:\"access_token\" OR resp.raw.cont:\"refresh_token\")",
    "serialization": {
        "regex_excerpt": {
            "regex": "(?:Authorization|authorization):\\s*Bearer\\s+[A-Za-z0-9._~+\\-/=]+|(?:X-API-Key|x-api-key):\\s*[^\\r\\n]+|\"(?:access_token|refresh_token|api_token)\"\\s*:\\s*\"[^\"]+\"",
            "context_chars": 32
        }
    },
    "fields": [
        "id",
        "request.method",
        "request.url",
        "response.status_code",
        "match_context.excerpts"
    ]
}
```

Extract only CSRF/session-like fragments from exact requests:

```json
{
    "ids": [123, 124],
    "serialization": {
        "regex_excerpt": {
            "regex": "\"(?:csrf|csrf_token|session|session_id)\"\\s*:\\s*\"[^\"]+\"|(?:csrf|session_id)=[^&\\s\";]+",
            "context_chars": 24
        }
    },
    "fields": ["id", "request.url", "match_context.excerpts"]
}
```

Remember: `regex_excerpt` extracts snippets from rows already selected by the tool. Use HTTPQL to narrow the row set first.

## Sitemap

Use Sitemap when the user needs Caido's deduplicated host/path tree instead of chronological history.

Root discovery:

```json
{
    "limit": 50,
    "fields": ["id", "label", "kind", "metadata", "requests.count"]
}
```

Entry lists use offset pagination:

```json
{
    "offset": 50,
    "limit": 50,
    "fields": ["id", "label", "kind", "requests.count"]
}
```

Continue while `has_more=true` using `next_offset`.

Subtree search:

```json
{
    "parent_id": 1,
    "depth": "ALL",
    "limit": 100,
    "filter": { "kinds": ["REQUEST"], "label_regex": "login|auth|api" },
    "fields": ["id", "label", "kind", "parent_id", "requests.count"]
}
```

Sample requests attached to entries:

```json
{
    "parent_id": 1,
    "depth": "DIRECT",
    "request_limit": 2,
    "fields": ["id", "label", "kind", "requests.count", "requests.items.id", "requests.items.url"]
}
```

For real request pagination under one entry, use `list_sitemap_entry_requests`:

```json
{
    "entry_id": 1,
    "limit": 50,
    "fields": ["cursor", "id", "request.method", "request.url", "response.status_code"]
}
```

Continue with `page_info.end_cursor`. This tool supports the same HTTP serialization controls as history for concrete request bodies, raw messages, and `regex_excerpt`.

Example: return only token-like snippets from requests under one Sitemap entry:

```json
{
    "entry_id": 1,
    "limit": 25,
    "serialization": {
        "regex_excerpt": {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vvvvvvvvvvel/VibeHacking](https://github.com/vvvvvvvvvvel/VibeHacking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
