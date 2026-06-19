---
trigger: always_on
description: Guidance for AI coding agents (Claude Code, Codex, Cursor, Copilot, …) working in this repository.
---

# AGENTS.md

Guidance for AI coding agents (Claude Code, Codex, Cursor, Copilot, …) working in this repository.
Single source of truth — `CLAUDE.md` imports this file. Keep it terse: task→files + ≤1 gotcha per
row; verbose details and live-verified behaviors live in [docs/dev-guide.md](docs/dev-guide.md)
(not auto-loaded — read the matching row there when working on one of these tasks).

## Project Overview

**ARC-1** is a TypeScript MCP (Model Context Protocol) server for SAP ABAP Development Tools (ADT).
It provides 12 intent-based tools (SAPRead, SAPSearch, SAPWrite, SAPActivate, SAPNavigate, SAPQuery,
SAPTransport, SAPGit, SAPContext, SAPLint, SAPDiagnose, SAPManage) for Claude and other MCP clients.
Distributed as npm package (`arc-1`) and Docker image (`ghcr.io/arc-mcp/arc-1`).

## Design Principles

1. **Centralized admin control** — managed service; server-wide safety ceiling (`allowWrites`, package allowlists, SQL/data/transport/Git gates, deny actions); every call audited; per-user scopes restrict, never expand.
2. **Per-user SAP identity** — principal propagation maps each MCP user to their own SAP user (BTP Destination Service + Cloud Connector); SAP auth applies per user.
3. **Token-efficient tools** — 12 intent tools vs 200+ endpoints, with schema payload guarded by CI budgets; hyperfocused mode = 1 tool (~200 tokens); method-level surgery + context compression keep mid-tier LLMs viable.
4. **BTP-native deployment** — Destination Service, Cloud Connector, XSUAA OAuth, BTP Audit Log; also Docker/npm/stdio.
5. **Multi-client, vendor-neutral** — XSUAA OAuth + Entra ID OIDC + API key coexist; one instance serves Claude, Copilot Studio, VS Code, Gemini CLI, Cursor.
6. **Safe defaults, opt-in power** — read-only by default; free SQL blocked; package allowlist defaults to `$TMP`; everything forbidden until the admin allows it.

## Build & Test

```bash
npm ci                          # Install dependencies
npm run build                   # TypeScript → dist/ (also copies AFF schemas)
npm test                        # Unit tests (all)
npx vitest run tests/unit/adt/client.test.ts   # Single test file
npx vitest run -t "getProgram"  # Tests matching a name pattern
npm run typecheck               # tsc --noEmit (src + scripts + tests via tsconfig.tests.json)
npm run lint / lint:fix / format  # Biome
npm run dev / dev:http          # Dev mode (stdio / HTTP Streamable)
npm run test:integration[:slow|:crud]  # Needs SAP credentials (TEST_SAP_URL)
npm run test:e2e[:slow]         # Needs running MCP server (syncs fixtures first)
TEST_BTP_SERVICE_KEY_FILE=~/.config/arc-1/btp-abap-service-key.json npm run test:integration:btp[:smoke]
```

Pre-commit: Husky runs `lint-staged` → Biome auto-fixes staged `*.{ts,js,json}`. Never hand-fix formatting.

## Configuration (Priority: CLI > Env > .env > Defaults)

Copy `.env.example` to `.env`. Parser: `src/server/config.ts`; defaults: `src/server/types.ts`.
Full per-option details (defaults, clamps, layer interactions): [docs_page/configuration-reference.md](docs_page/configuration-reference.md).

| Variable / Flag | Description |
|-----------------|-------------|
| `SAP_URL`, `SAP_USER`, `SAP_PASSWORD`, `SAP_CLIENT` | SAP connection (client default 100) |
| `SAP_LANGUAGE` | Request language AND master language of created objects (default EN, #343) |
| `SAP_INSECURE` | Skip TLS verification (default false) |
| `SAP_TRANSPORT` | `stdio` (default) or `http-streamable` |
| `ARC1_PORT` / `ARC1_HTTP_ADDR` | HTTP port (8080) / full bind address |
| `SAP_ALLOW_WRITES` | Enable mutations (default false); prerequisite for transport/git writes |
| `SAP_ALLOW_DATA_PREVIEW` / `SAP_ALLOW_FREE_SQL` | TABLE_CONTENTS preview / freestyle SQL (default false) |
| `SAP_ALLOW_TRANSPORT_WRITES` / `SAP_ALLOW_GIT_WRITES` | Transport / git mutations (each ALSO needs `SAP_ALLOW_WRITES`) |
| `SAP_ALLOWED_PACKAGES` | Write allowlist (default `$TMP`): exact, `Z*`, `ZFOO/**` subtree, `*`. Enforced fail-closed on every mutation incl. activation, against the object's REAL package |
| `SAP_DENY_ACTIONS` | Per-action denial: `Tool`, `Tool.action`, `Tool.glob*` — see docs_page/authorization.md |
| `ARC1_API_KEYS` | `key:profile` pairs (viewer…admin); profile ∩ server ceiling |
| `SAP_OIDC_ISSUER` / `SAP_OIDC_AUDIENCE` | OIDC JWT validation |
| `ARC1_OAUTH_DCR_TTL_SECONDS` | DCR client_id lifetime (default 30d; `0` = no expiry for clients that don't re-register) |
| `ARC1_DCR_SIGNING_SECRET` | Dedicated HMAC secret so `cf deploy` doesn't invalidate cached client_ids |
| `ARC1_ALLOWED_ORIGINS` | CORS allowlist for browser MCP clients (empty = CORS off) |
| `ARC1_PUBLIC_URL` | Advertised OAuth-metadata URL when behind a reverse proxy |
| `SAP_BTP_SERVICE_KEY[_FILE]` / `SAP_BTP_OAUTH_CALLBACK_PORT` | BTP ABAP service key / OAuth callback port |
| `SAP_SYSTEM_TYPE` | `auto` (default), `btp`, `onprem` |
| `SAP_ABAP_RELEASE` | SAP_BASIS release override for abaplint (e.g. 758, 816); probe wins |
| `ARC1_TOOL_MODE` | `standard` (12 tools) or `hyperfocused` (1 tool, ~200 tokens) |
| `ARC1_PLUGINS` | FEAT-61 extensions: CSV of absolute LOCAL paths (`.js`/`.json`), NOT npm. Adds `Custom_*` tools (read-only v1) — docs_page/extensions.md |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arc-mcp/arc-1](https://github.com/arc-mcp/arc-1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
