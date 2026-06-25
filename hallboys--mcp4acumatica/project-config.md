---
trigger: always_on
description: Remote MCP (Model Context Protocol) server on Cloudflare Workers that connects Claude to an Acumatica ERP 2025 R2 instance via the contract-based REST API. Each user authenticates directly with Acumatica — their Acumatica role controls what records they can access.
---

# CLAUDE.md — Project Memory for MCP4Acumatica

## Project Overview

Remote MCP (Model Context Protocol) server on Cloudflare Workers that connects Claude to an Acumatica ERP 2025 R2 instance via the contract-based REST API. Each user authenticates directly with Acumatica — their Acumatica role controls what records they can access.

- **License:** Apache 2.0 — Copyright 2026 Hall Boys, Inc.
- **Copyright header** required on all `.ts` source files: `// Copyright 2026 Hall Boys, Inc.` + `// SPDX-License-Identifier: Apache-2.0`
- **Git config (this repo only):** `user.email = saratvemuri@hallboys.com`
- **Current tag:** `25R2-0.36.0`
- **Deployed at:** `https://mcp4acumatica.hallboys.com` (primary custom domain) / `https://acumatica-mcp.hallboys.com` (legacy alias, kept active during migration) / `https://mcp4acumatica.<account>.workers.dev` (workers.dev fallback)
- **GitHub:** `https://github.com/hallboys/MCP4Acumatica`

## Architecture

```
Claude (claude.ai / Desktop / API)
    │
    ▼  MCP over streamable-http
┌─────────────────────────────────┐
│  Cloudflare Worker              │
│  OAuthProvider wrapper          │
│    ├─ /authorize → Acumatica    │
│    ├─ /callback  ← Acumatica   │
│    ├─ /token, /register (DCR+CIMD) │
│    ├─ /docs → Documentation site │
│    └─ /mcp → McpAgent DO        │
│       ├─ 48 tools (38 read-only  │
│       │   + 6 utility/discovery  │
│       │   + 4 schema-knowledge)  │
└──────────────┬──────────────────┘
               │  Bearer token (per-user)
               ▼
        Acumatica 25R2 SaaS
        Contract-Based REST API
        Default/25.200.001
```

### Storage Abstraction (Platform Portability)

Tool handlers, the Acumatica HTTP client, config, and caching are decoupled from Cloudflare via two abstractions:

- **`IKeyValueStore`** (`src/lib/kv-store.ts`) — Platform-agnostic interface for key-value storage (get, put, delete, list). Cloudflare Workers uses `CloudflareKVStore` which wraps `KVNamespace`.
- **`AppEnv`** (`src/types/acumatica.ts`) — Portable environment type containing Acumatica connection settings and a `store: IKeyValueStore`. All tool handlers and shared libraries use `AppEnv`. The Cloudflare-specific `Env` extends `AppEnv` with CF bindings (`TOKEN_STORE`, `OAUTH_KV`, `MCP_OBJECT`, etc.).

This design allows future self-hosted adapters (Node.js + Redis/SQLite) to reuse all tool handlers without modification. See `docs/self-hosting-guide.md`.

## OAuth Flow

Claude → Worker `/authorize` → Acumatica login (with `openid profile email api offline_access` scopes) → Worker `/callback` → OIDC userinfo → canary GI role check → `/consent` interstitial → token stored → MCP session active.

Acumatica is the sole identity provider. Users log in with their Acumatica credentials (or via whatever SSO their Acumatica instance is configured with). The MCP server does not manage identity separately — it delegates entirely to Acumatica.

### Access Control & Governance

1. **Role gate (canary GI):** After login, the callback queries the `MCPAccess` Generic Inquiry via OData. This GI is assigned only to the `MCP Access` role in Acumatica. If the OData query returns 200, the user has the role; if 403, they don't. This avoids exposing user/role data — the GI content is irrelevant, it's purely an access gate. Users without the role see a 403 page directing them to contact their Acumatica admin. The role name is configurable via `ACUMATICA_MCP_ROLE` env var.

2. **Consent interstitial:** Users who pass the role check see a consent page explaining that data will be processed by AI, access is logged, and sensitive fields are redacted. They must acknowledge before the MCP session activates.

3. **Sensitive field redaction:** Tool responses are automatically scanned for sensitive field names (SSN, bank accounts, salary, credit card, etc.) using pattern matching. Matched values are replaced with `[REDACTED]`. Patterns are configurable via `REDACT_PATTERNS` (add) and `REDACT_SKIP` (whitelist) env vars. See `src/lib/redact.ts`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hallboys/MCP4Acumatica](https://github.com/hallboys/MCP4Acumatica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
