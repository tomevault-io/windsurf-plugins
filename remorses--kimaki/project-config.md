---
trigger: always_on
description: after every change always run tsc inside cli to validate your changes. try to never use as any
---

after every change always run tsc inside cli to validate your changes. try to never use as any

do not use spawnSync. use our util execAsync. which uses spawn under the hood

the important package in this repo is cli. it contains the discord bot code.

after making important changes to queueing or message handling always run the full test suite inside cli to make sure our changes did not break anything. also run with -u and see snapshots updates in git diff if needed. `pnpm test -u --run`

# repo architecture

kimaki is a monorepo with three main packages that communicate via a shared Postgres database hosted on PlanetScale.

```
┌─────────────────────────────────────────────────────────────┐
│  User's machine                                             │
│  cli/ (TypeScript CLI + Discord bot)                        │
│  ├── src/cli.ts        main CLI, onboarding wizard          │
│  ├── src/discord-bot.ts  event loop, session routing        │
│  └── SQLite (~/.kimaki/discord-sessions.db)                 │
│         local state: bot tokens, channels, threads, models  │
└────────┬──────────────────────────┬─────────────────────────┘
         │ REST + WebSocket         │ polls /api/onboarding/status
         │ (clientId:secret)        │ during first-time setup
         ▼                          ▼
┌─────────────────────┐   ┌──────────────────────────────────┐
│  gateway-proxy/      │   │  website/                        │
│  (Rust, fly.io)      │   │  (Cloudflare Worker, Hono)       │
│                      │   │  https://kimaki.dev           │
│  Sits between the    │   │                                  │
│  CLI and Discord.    │   │  GET /oauth/callback              │
│  One shared bot for  │   │    → upserts gateway_clients row │
│  all users — users   │   │    → website/src/routes/          │
│  don't create their  │   │      oauth-callback.tsx           │
│  own Discord bot.    │   │                                  │
│                      │   │  GET /api/onboarding/status       │
│  Multi-tenant:       │   │    → CLI polls every 2s           │
│  filters events per  │   │    → website/src/routes/          │
│  client_id + guild   │   │      onboarding-status.ts         │
│                      │   │                                  │
│  wss://kimaki-       │   └──────────┬───────────────────────┘
│  gateway-production  │              │
│  .fly.dev            │              │
└──────────┬───────────┘              │
           │                          │
           ▼                          ▼
┌──────────────────────────────────────────────────────────────┐
│  Shared Postgres (PlanetScale)                               │
│  db/schema.prisma                                            │
│                                                              │
│  gateway_clients table:                                      │
│    client_id  TEXT   ── identifies the kimaki user            │
│    secret     TEXT   ── authenticates gateway connections     │
│    guild_id   TEXT   ── guild the user installed the bot in   │
│    @@id([client_id, guild_id])                               │
│                                                              │
│  Written by: website (on OAuth callback)                     │
│  Read by: gateway-proxy (polls every 1s via db_config.rs)    │
│  Read by: website (onboarding status check)                  │
└──────────────────────────────────────────────────────────────┘
```

## gateway-proxy (Rust)

`gateway-proxy/` is a Rust service that proxies both Discord Gateway (WebSocket) and REST traffic. it lets multiple users share a single Discord bot instead of each user creating their own.

key files:

- `src/main.rs` — entry point, shard setup, HTTP server, DB polling
- `src/auth.rs` — authenticates `client_id:secret` tokens
- `src/db_config.rs` — polls Postgres `gateway_clients` table every 1s, atomically swaps the in-memory client map. stale protection: rejects auth if DB unreachable >30s
- `src/server.rs` — HTTP+WS server. REST proxy at `/api/v10/*`, WebSocket upgrade for gateway
- `src/dispatch.rs` — per-shard event fanout, filters events by `authorized_guilds`
- `src/cache.rs` — builds synthetic READY payloads filtered to authorized guilds
- `src/rest_proxy.rs` — forwards REST calls, rewrites Authorization header to real bot token, scopes guild/channel routes

auth flow: client sends IDENTIFY with token `client_id:client_secret` → proxy validates against the CLIENTS map (from DB) → returns `SessionPrincipal::Client(id)` + `authorized_guilds` → only forwards events for those guilds.

gateway REST rule for cli package code: when running with `client_id:secret`
through gateway-proxy, Discord REST calls must be guild-scoped or explicitly
allowlisted by the proxy (`/gateway/bot`, `/users/@me`, etc). avoid global
application routes like `/applications/{app_id}/commands`; use
`/applications/{app_id}/guilds/{guild_id}/commands` instead so auth can resolve
scope and allow the request.

multi-tenant REST safety invariant:

- never allow client-authenticated requests to hit unscoped bot-token routes.
- only tokenized interaction/webhook routes are allowed without auth
  (`/interactions/{id}/{token}/...`, `/webhooks/{id}/{token}/...`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [remorses/kimaki](https://github.com/remorses/kimaki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
