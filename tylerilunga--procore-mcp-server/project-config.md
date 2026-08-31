---
trigger: always_on
description: > MCP server exposing the full Procore REST API for Claude Desktop and Claude Code. Single-user OAuth. TypeScript + @modelcontextprotocol/sdk.
---

# Procore MCP Server

> MCP server exposing the full Procore REST API for Claude Desktop and Claude Code. Single-user OAuth. TypeScript + @modelcontextprotocol/sdk.

## Quick Start

```bash
npm install
npm run build          # Generate catalog from OAS + compile TypeScript
npm run auth           # One-time: OAuth flow to get Procore tokens
npm start              # Start MCP server (stdio transport)
```

## Architecture

**7 MCP tools** provide full coverage of Procore API endpoints:

| Tool | Purpose |
|------|---------|
| `procore_discover_categories` | List API categories with endpoint counts |
| `procore_discover_endpoints` | List endpoints in a category/module |
| `procore_get_endpoint_details` | Get full parameter schema for an endpoint |
| `procore_api_call` | Execute any Procore API call |
| `procore_search_endpoints` | Full-text search across endpoints |
| `procore_get_config` | Show current config (company_id, auth status) |
| `procore_set_config` | Set runtime config (company_id, project_id) |

### Build Pipeline

`specs/combined_OAS.json` (~54MB) -> `scripts/generate-catalog.ts` -> `data/catalog.json` + `data/endpoint-details/` -> `scripts/generate-tools-manifest.ts` -> `data/tools-manifest.json`

Current spec (2026-08-04): 3,155 operations -> 2,929 generated tools + 7 meta tools.
The manifest drops older-version duplicates of the same path and the
non-callable `/oauth/*` endpoints; both stay reachable via `procore_api_call`.

### Tool Description Quality

Generated tool descriptions are assembled from one sentence per scoring
dimension, and no sentence may restate another:

| Module | Responsibility |
|--------|----------------|
| `src/tools/resource-label.ts` | Names the actual resource from the OAS summary (never the category) |
| `src/tools/purpose-builder.ts` | Verb-aware purpose synthesis (a reorder is never described as a create) |
| `src/tools/description-builder.ts` | Deprecation notice, usage guidance, prerequisites, assembly |
| `src/tools/behavior-builder.ts` | Return shape, side effects, failure modes |
| `src/tools/param-descriptions.ts` | Per-parameter prose and source hints |
| `src/tools/annotation-builder.ts` | Titles and MCP annotations |
| `src/tools/version-sibling-note.ts` | Names cross-version sibling tools in the description |

Pagination is advertised only when the endpoint genuinely returns a collection
(`returnsCollection`), in both the description and the input schema.
`schemaIsCollection` in `scripts/generate-catalog.ts` unwraps `{ data: [...] }`
envelopes (and any single-property object wrapping an array, whatever it's
named — Procore's v1.0 endpoints often envelope under the plural resource
name, e.g. `{ exchange_rates: [...] }`) and merges `allOf` branches into one
schema before judging it, rather than treating `allOf` like `oneOf` (an
optional array-typed field on one branch doesn't make the *merged* object a
list). When the schema itself asserts a single object, that verdict only
flips on strong corroboration: a non-`{id}` path trusts either declared
pagination or explicit list language in the summary/description; an
`{id}`-shaped path requires *both*, since Procore sometimes declares
`page`/`per_page` on a genuine show-by-id endpoint with no textual list
signal behind it. A scalar/binary response (`type: "string", format:
"binary"` — a raw CSV/PDF file download) is never a collection regardless of
declared pagination, since a file body can't be "a JSON array of records".
The envelope key is recorded as `collectionEnvelope` and named in the
behavior sentence.

Procore's own OAS text is untrustworthy in two specific, narrow cases, both
handled in `generate-tools-manifest.ts` by clearing the inherited
`description` so the tool falls back to its own summary-driven synthesis:
a GET whose description opens with "Creates"/"Create" (a genuine spec error,
unlike a PATCH/PUT upsert legitimately described that way), and a
sub-resource action that shares byte-identical description text with its
base resource's endpoint (`signature_requests` vs
`signature_requests/{id}/signature`) — detected by one path's real segments
being a strict prefix of the other's, as opposed to a scope variant that
inserts a segment in the middle and legitimately shares one description.

Three rules keep the prose honest, all enforced by `npm test`
(`scripts/verify-manifest.ts`, which also runs in CI):

- **Never claim semantics the name does not carry.** A POST named `reorder_*`
  is not a create, a DELETE named `recycle_*` is a soft delete, and a
  `bulk_*`/`sync_*` call is described in the plural.
- **Name the right record.** The id closing a path is the *target*, not a
  "parent record"; non-identifier path params (`{new_status}`) are neither.
  A multipart/form-data PATCH/PUT (a file upload) doesn't get the "send only
  the fields you intend to change" partial-update clause — the whole file is
  what's being replaced, not a selection of JSON fields.
- **Disambiguate cross-version siblings.** When Procore exposes the same
  operation at two API versions under different paths (dedup can't merge
  them — e.g. v1.1 moved weather logs under `daily_logs/`), each tool's

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TylerIlunga/procore-mcp-server](https://github.com/TylerIlunga/procore-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
