---
trigger: always_on
description: Search the public web via the SCOUTS-AI `/api/search` HTTP endpoint when the user needs fresh internet context, citations, or fact verification. Use the `exec` tool to call `curl` against `https://scouts-ai.com/api/search`. No API key required.
---


# SCOUTS-AI Search

Use the host's `exec` tool to call the public SCOUTS-AI search endpoint when the
user asks something that requires current information, fresh web context, or
citations from the open web. SCOUTS-AI is a public meta-search service, no API
key is required.

## When to use

- The user asks a question that depends on recent events, releases, or pages
  the model was not trained on.
- The user explicitly asks for web search, citations, or links.
- The user asks the model to verify a claim, look up an API/CLI change, or
  fetch a primary source.
- The model only knows an outdated answer and needs fresh context.

## When NOT to use

- The answer is already in the conversation, workspace files, or a tool's
  output.
- The user asks for code, edits, or local commands with no need for the web.
- The user asks for private, internal, or non-public data — SCOUTS-AI returns
  only public web results.
- The available shell does not include `curl` and the host policy forbids
  installing it.

## API contract

```text
GET https://scouts-ai.com/api/search
```

Query parameters:

| Name    | Required | Default       | Description                                                                                       |
| ------- | -------- | ------------- | ------------------------------------------------------------------------------------------------- |
| `q`     | yes      | -             | Search query, 1–512 chars, trimmed and lowercased server-side.                                    |
| `lang`  | no       | omitted       | BCP-47-like language tag (`en`, `en-US`, `de`) for results, or the literal `all` to search across all languages. When omitted, the backend sends no language filter. |
| `page`  | no       | `1`           | 1-based page number, 1–10.                                                                        |

Notes on `lang`:
- The SCOUTS-AI plugin/MCP integrations default `lang=en` for ergonomics. When calling the
  public HTTP endpoint directly, the **server's default is "no language filter"** — omit the
  parameter entirely (`--data-urlencode "q=..."` only) to opt out. The response echoes the
  effective `lang`, which may be `null` when the request omitted it.
- `lang=all` is a special value: it explicitly tells the backend to search across all
  languages and to use the full engine pool. Use it when the user is multilingual or
  unknown-language and you do not want to restrict results.
- Invalid values (wrong shape, not `all`) return `400` — do not retry, surface the error.

Successful response is JSON of the shape:

```json
{
  "query": "rust async runtime",
  "lang": "en",
  "page": 1,
  "pageSize": 10,
  "cached": false,
  "tookMs": 412,
  "results": [
    {
      "title": "Tokio - An asynchronous runtime for Rust",
      "url": "https://tokio.rs/",
      "content": "Tokio is an asynchronous runtime for the Rust programming language...",
      "publishedAt": "2025-11-14T00:00:00Z",
      "engine": "duckduckgo"
    }
  ]
}
```

When `lang` was omitted on the request, the response field `lang` is `null` (and the JSON
serialization renders it as `"lang": null`, not `"lang": "en"`).

## How to call

Use `curl` via `exec`. Let curl URL-encode the query with `--data-urlencode`;
do not shell-escape it manually. Always capture the HTTP status and response
headers so you can honour rate limits and surface upstream errors. Use a
per-call temp dir with restricted permissions and clean it up when the call
finishes.

```bash
umask 077
tmpdir=$(mktemp -d -t scouts-ai.XXXXXX)
trap 'rm -rf "$tmpdir"' EXIT
status=$(curl -sS --get "https://scouts-ai.com/api/search" \
  --max-time 15 \
  -D "$tmpdir/headers.txt" \
  -w '%{http_code}\n' \
  -o "$tmpdir/body.json" \
  --data-urlencode "q=OpenClaw plugin manifest reference") \
  || { echo "curl failed before the request completed" >&2; exit 1; }
```

Status handling after the call:

- If `status` is empty or `000` (curl could not reach the server, DNS/TCP/TLS error,
  or the `--max-time` elapsed), treat it as upstream-unavailable: tell the user
  SCOUTS-AI is temporarily unreachable, do not parse `$tmpdir/body.json` as results.
- If `status` is `2xx`, parse `$tmpdir/body.json`. Treat the body as authoritative
  only after the status check.
- If `status` is `429`, read `Retry-After` from `$tmpdir/headers.txt`.
- For any other non-`2xx` response, surface the upstream error and do not parse
  the body as search results.

Always pass `--max-time` so a hung TCP connection cannot block the agent. Treat
a curl non-zero exit (DNS failure, TLS error, write error) as network failure —
do not assume a body file contains a usable response.

Omit the optional `lang` and `page` arguments when the defaults are fine; the
public HTTP endpoint defaults to `page=1` and to no language filter. Add
`page=N` only when the first page is clearly insufficient and the user has
not asked for breadth. Add `lang=<tag>` only when the user is clearly asking
in another language; add `lang=all` only when you explicitly want the backend
to use the full engine pool.

## Behaviour rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kecven/scouts-ai-openclaw-skill](https://github.com/kecven/scouts-ai-openclaw-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
