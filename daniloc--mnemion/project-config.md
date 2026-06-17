---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Mnemion

Persistent, evolving shared memory between a human and their AI agents. MCP server on Cloudflare Workers.

## Project structure

```
project-docs/active/   Design documents (the "why" and "what")
mnemion-js/            Cloudflare Worker — MCP server (the "how")
  src/index.ts         Route table + OAuthProvider config (~60 lines)
  src/router.ts        Declarative router: types, enums, pattern matching, dispatch
  src/routes/auth.ts   /authorize, /auth/verify, passkey setup & authentication, /invite/:token approval
  src/routes/io.ts     /o/entry/:pattern/:id shared entries, /o/:path egress, /p/:path publications, /f/:token|:id documents, /i/:path ingress, /upload/:token
  src/routes/marketplace.ts  Token management, git endpoints (composes query + git adapter)
  src/routes/dev.ts    Dev-only seed routes (Auth.DEV gated, inert in production)
  src/routes/canvas.ts /canvas SSR page + /api/canvases, /api/canvas (list/save tldraw snapshots)
  src/routes/pages.ts  Svelte SSR pages, /api/* JSON endpoints, WebSocket proxy
  src/session.ts       SessionDO: McpAgent, MCP protocol handler (per-session DO)
  src/hive.ts          HiveDO: DO shell — RPC wrappers, URI resolution, federation, WebSocket
  src/data.ts          Query engine, mutation engine (CRUD), cross-pattern search
  src/evolution.ts     Schema evolution: CHANGE_TYPES declaration table, propose/apply/revert
  src/prime.ts         Auto-associative priming: Workers AI embeddings + Vectorize KNN
  src/publications.ts  Publication renderers: live pattern data → HTML/RSS/JSON/markdown, template seam
  src/web.ts           Web URL resolution adapter dispatch (Bluesky, browser-rendering); _web_cache
  src/tools.ts         Tool metadata SSOT — feeds session.ts MCP registration and /api/tools
  src/credentials.ts   Passkey CRUD, access token validation
  src/kernel.ts        Pre-mutation hooks, immutable fields, scope matching
  src/labels.ts        Single source of truth for "what does this entry look like" (deriveLabel, truncate)
  src/schema.ts        DDL, migrations, kernel table declarations, system doc seeding
  src/dev-seed.ts      DEV_SEED-gated realistic data population (raw SQL, runs in DO ctor)
  src/passkey.ts       WebAuthn passkey registration + authentication
  src/transform.ts     Transform DSL evaluator for ingress field mapping
  src/git.ts           Git protocol adapter (file tree → git pack, used by marketplace)
  src/constants.ts     Product identity (PRODUCT_NAME, URI_SCHEME, uri() helper)
  src/system-docs/     Markdown files with {{placeholder}} syntax, loaded at runtime
  src/pages/           Svelte components (SchemaViewer, HiveMap, LinkMap, Canvas, EntryDetail) + SSR + canvas entry points
  vite.config.ts       Main SSR + client build (SchemaViewer/HiveMap/LinkMap/EntryDetail)
  vite.canvas.ts       Separate build for canvas-client.client.txt → dist/canvas/
  scripts/setup.sh     First-run setup: generates secret, deploys, opens passkey registration
```

Future peer: iOS app (Swift).

## Current state

Deploys as a single Cloudflare Worker that exposes MCP at `/mcp`. Cross-surface data sharing proven (Claude Code + Claude.ai reading/writing the same store).

### Architecture

Two Durable Objects:
- **SessionDO** (`McpAgent`) — one per MCP session, handles protocol, proxies to hive via RPC
- **HiveDO** — one per user, holds all SQLite data. Keyed by `user:{userId}` (currently always `"owner"`)

HiveDO is a thin wiring shell. Domain logic lives in pure-function modules with `db`/context injected:
- **`data.ts`** — query, mutate, search (DataContext)
- **`evolution.ts`** — schema evolution via `CHANGE_TYPES` declaration table (EvolutionContext)
- **`credentials.ts`** — passkey + token operations (db)
- **`kernel.ts`** — pre-mutation hooks, immutable fields, scope matching
- **`schema.ts`** — DDL, migrations, kernel table declarations, audit triggers

### Auth

Layered auth behind OAuth 2.1. The `workers-oauth-provider` package wraps the worker and handles the OAuth flow (DCR, tokens).

- **Master secret** (`MNEMION_SECRET`): High-entropy random hex, set via `npm run setup`. Root of trust — used to bootstrap the owner passkey and as fallback for headless agents. The user's Cloudflare login is the true credential; the secret is ephemeral and replaceable. Master-secret logins resolve to the `owner` actor.
- **Passkey** (WebAuthn): Optional convenience layer for browser-based OAuth. Registered via one-time setup URL. Stored in HiveDO `_passkeys` table — **one credential per member** (re-registering a member replaces only that member's row). If any passkey is registered, `/authorize` shows passkey-first UI with secret fallback. Authentication offers all members' credentials and resolves the actor from the one used.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daniloc/mnemion](https://github.com/daniloc/mnemion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
