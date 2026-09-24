---
trigger: always_on
description: `feishu-docs-cli` is a Node.js CLI tool for AI Agents to read/write Feishu (Lark) cloud documents and knowledge bases via shell commands. It outputs pure text or structured JSON — no interactive UI.
---

# AGENTS.md

## Project Overview

`feishu-docs-cli` is a Node.js CLI tool for AI Agents to read/write Feishu (Lark) cloud documents and knowledge bases via shell commands. It outputs pure text or structured JSON — no interactive UI.

> [!IMPORTANT]
> **Zero runtime dependencies**: All API calls use `fetchWithAuth()` in `src/client.ts` with native `fetch`. No external SDK is used.

> [!IMPORTANT]
> **Convert + Descendant API for writing**: Document content is written using the two-step Convert API (`POST /docx/v1/documents/blocks/convert`) + Descendant API (`POST /docx/v1/documents/{id}/blocks/{id}/descendant`). Do NOT use the children API (`POST /docx/v1/documents/{id}/blocks/{id}/children`) which has a 9-row table limit. Do NOT implement a local Markdown parser — the server-side conversion handles all formatting.

## Build & Test

```bash
npm install            # Install dependencies
npm test               # Run all tests (node:test)
node bin/feishu-docs.js --help   # Run CLI
```

> [!IMPORTANT]
> **Test Coverage**: All new features must include unit tests. Run `npm test` and verify 0 failures before committing. Tests use Node.js built-in `node:test` and `node:assert/strict` — no test frameworks.

## Architecture

### Source Layout

| File | Purpose |
|------|---------|
| `src/cli.ts` | Entry point, declarative command routing with subcommand support |
| `src/auth.ts` | OAuth v2 login flow, encrypted token persistence, file-lock refresh |
| `src/client.ts` | Auth client factory, `fetchWithAuth()` REST wrapper, `fetchBinaryWithAuth()` for binary responses, token resolution |
| `src/commands/*.ts` | Command modules — each exports `meta` for routing |
| `src/services/block-writer.ts` | Document backup, clear, restore helpers |
| `src/services/wiki-nodes.ts` | Wiki node tree traversal and token resolution |
| `src/services/doc-blocks.ts` | Document block fetching with pagination |
| `src/services/markdown-convert.ts` | Convert API + Descendant API pipeline |
| `src/services/doc-enrichment.ts` | Sheet/bitable/whiteboard/image enrichment with parallel execution |
| `src/services/image-download.ts` | Image download and local cache with TTL eviction |
| `src/parser/blocks-to-md.ts` | Feishu block JSON → Markdown renderer |
| `src/parser/block-types.ts` | Block type constants |
| `src/parser/text-elements.ts` | Inline text element rendering |
| `src/scopes.ts` | OAuth scope catalog: BASE_SCOPES (免审), mergeScopes, buildScopeHint |
| `src/utils/errors.ts` | `CliError` class, `mapApiError()`, exit codes |
| `src/utils/url-parser.ts` | URL/token parsing and validation |
| `src/utils/document-resolver.ts` | Unified URL/token → document descriptor resolution |
| `src/utils/member.ts` | Shared member ID validation and type detection |
| `src/utils/drive-types.ts` | Document type → Drive API type mapping |
| `src/utils/validate.ts` | Token/ID format validation for path safety |
| `src/utils/version.ts` | Local version read, npm update check (24h cache, non-blocking) |
| `src/utils/retry.ts` | Exponential backoff with jitter for API retries |
| `src/utils/concurrency.ts` | Zero-dependency `pLimit()` concurrency limiter |

### Command Registration Pattern

Each command module exports a `meta` object:

```javascript
// Top-level command
export const meta = {
  options: { raw: { type: "boolean" }, blocks: { type: "boolean" } },
  positionals: true,
  handler: read,
};

// Command with subcommands
export const meta = {
  subcommands: {
    list: { options: {}, positionals: true, handler: list },
    add:  { options: { role: { type: "string" } }, positionals: true, handler: add },
  },
};
```

Register in `src/cli.js` by importing the meta and adding to the `COMMANDS` object.

### Auth Modes

| Mode | Token Type | When to Use |
|------|-----------|-------------|
| `user` | user_access_token | Personal docs, collaboration, search, wiki member management |
| `tenant` | tenant_access_token | App-managed docs, CI/CD |
| `auto` | Best available | Default — tries user first, falls back to tenant |

## Input Validation & Path Safety

> [!IMPORTANT]
> This CLI is designed to be invoked by AI agents. Always assume inputs can be adversarial — validate all user-supplied values before embedding them in API URL paths.

### Token/ID Validation (`src/utils/validate.js`)

All tokens, space IDs, and node tokens interpolated into URL paths MUST be validated:

```javascript
import { validateToken } from "../utils/validate.js";

validateToken(spaceId, "space_id");  // Throws CliError if invalid
```

### URL Path Encoding

All dynamic segments in URL paths MUST use `encodeURIComponent`:

```javascript
// CORRECT
`/open-apis/wiki/v2/spaces/${encodeURIComponent(spaceId)}/members`

// WRONG — raw user input in URL path
`/open-apis/wiki/v2/spaces/${spaceId}/members`
```

### Member ID Validation (`src/utils/member.js`)

User-supplied member IDs MUST be validated before use:

```javascript
import { validateMemberId, detectMemberType } from "../utils/member.js";

validateMemberId(memberId);  // Regex validation
const memberType = detectMemberType(memberId);  // Auto-detect: email, openid, unionid, etc.
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cliff-byte/feishu-docs-cli](https://github.com/cliff-byte/feishu-docs-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
