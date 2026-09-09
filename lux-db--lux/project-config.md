---
trigger: always_on
description: Lux is an application database written in Rust. One engine provides
---

# Lux

Lux is an application database written in Rust. One engine provides
Redis-compatible commands, typed tables, vectors, time series, streams,
realtime subscriptions, app authentication, row-level grants, and push
notifications. It exposes RESP for trusted infrastructure and versioned HTTP,
auth, and WebSocket APIs for applications.

Use this file as the short product map. The authoritative public contracts are
[README.md](README.md), [COMPATIBILITY.md](COMPATIBILITY.md),
[DURABILITY.md](DURABILITY.md), [SECURITY.md](SECURITY.md), and
[MANAGEMENT_API.md](MANAGEMENT_API.md).

## Start a local project

Docker is required for the local stack.

```bash
curl -fsSL https://luxdb.dev/install.sh | sh
lux init
lux start
```

`lux start` launches a real Lux engine and Lux Studio on loopback, applies
pending migrations, seeds a fresh data volume, and creates a private `local`
environment profile. On first setup it safely merges the Lux variables into
`.env.local`; unrelated variables and comments are preserved.

```bash
lux status             # local runtime and connection details
lux studio             # open the local Studio
lux doctor             # runtime, env, API, and migration checks
lux target             # local, linked-Cloud, and active app-env targets
lux stop               # keep local data
lux stop --clear       # delete the local data volume
lux start --fresh      # recreate the volume, then migrate and seed it
```

Local ports bind to `127.0.0.1` by default. Use `lux start --bind <IP>` only
when a trusted device or development environment must connect over the network.
Use `lux start --no-studio` when only the engine is needed.

## Connections and credentials

| Variable | Use |
|---|---|
| `LUX_URL` | HTTP base URL used by browser, server, and Swift project clients |
| `LUX_DIRECT_URL` | RESP connection URL for trusted servers, CLI commands, and Redis-compatible clients |
| `LUX_PUBLISHABLE_KEY` | Browser/mobile-safe project key; data access still requires a signed-in user and matching grants |
| `LUX_SECRET_KEY` | Trusted-server key with full project access; never ship it to a browser or mobile app |

Use `lux://` or `luxs://` URLs with Lux's CLI and direct TypeScript client.
Third-party Redis clients do not know those schemes, so use `redis://` or
`rediss://` with the same host and credential. Never expose a direct database
password to client code.

Use an HTTP project client for application tables, auth, grants, vectors, time
series, live queries, and push. Use RESP from trusted infrastructure for cache
and key/value workloads, collections, streams and queues, Pub/Sub, and direct
Lux commands. Both surfaces address the same engine; they are not separate
services.

An omitted CLI target means the local engine for migrations, seeds, types,
auth-provider configuration, and push configuration. A linked Cloud project is
used only when it is named or a comparison command such as `--all` asks for it.
`lux link` does not silently change local commands into production commands.

## Schema and migrations

Lux migrations are ordered UTF-8 command files under `lux/migrations/`.
Schema, indexes, grants, and stable seed-independent data belong in migrations.

```bash
lux migrate new create_messages
lux migrate plan
lux migrate run
lux migrate status --check
lux types
```

The engine owns parsing, SHA-256 identity, the migration ledger, command
progress, and repair state. Do not write `__migrations` directly. A failed or
interrupted migration blocks later migration writes and never resumes itself:
inspect `lux migrate status`, then use an explicit `lux migrate repair` action
only after reviewing the recorded command cursor.

`lux/seed.lux` uses the same command format but is not ledgered. Run it with
`lux seed run`; use stable identifiers when repeated seed execution must be
predictable.

## Lux Studio

Local Studio is part of the supported local stack, not a mock dashboard. It
talks directly to the engine and exposes engine-native Overview, command
editor, Migrations, Tables, Vectors, Queues, Time Series, Realtime, Backups,
Auth, Push, and Settings surfaces. It negotiates capabilities through `GET /v1`
instead of guessing from an image tag.

If Studio reports a compatibility problem, run `lux version`, `lux doctor`,
and the explicit update it recommends (`lux update engine` or
`lux update studio`). `lux start` never changes a running component's version
implicitly. Object storage, logs, domains, billing, and provisioning depend on
Cloud services and are not local Studio features.

## TypeScript applications

Install the stable first-party SDK:

```bash
bun add @luxdb/sdk
```

Use a publishable key in browser code and a secret key only on a trusted
server. Project clients return `{ data, error }`.

```ts
import { createBrowserClient } from "@luxdb/sdk/browser";

const lux = createBrowserClient(LUX_URL, LUX_PUBLISHABLE_KEY);

const { error: signInError } = await lux.auth.signInAnonymously();
if (signInError) throw signInError;

const { data: messages, error } = await lux
  .table("messages")
  .select()
  .eq("channel_id", "general")
  .order("created_at", { ascending: false })
  .limit(50);
if (error) throw error;
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lux-db/lux](https://github.com/lux-db/lux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
