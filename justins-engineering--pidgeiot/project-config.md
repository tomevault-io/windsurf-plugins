---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

PidgeIoT is an edge-native IoT platform built in Rust, structured as a Cargo workspace with three crates:

- **`dovecote`** (backend): serverless edge router on Cloudflare Workers + Durable Objects. Compiles to a `cdylib` via `worker-build`. Handles device ingestion, provisioning, and session validation.
- **`fancier`** (frontend): WebAssembly SPA built with Dioxus 0.7 + TailwindCSS/DaisyUI. The human-facing dashboard.
- **`capsules`** (shared models): serde structs/RPC schemas shared by both `dovecote` and `fancier` so frontend/backend stay in sync. Keep this crate free of Worker- or Dioxus-specific dependencies.

Auth/identity is handled by a self-hosted Ory Kratos instance for dashboard users (dev: `docker-compose.yml`; production since 2026-07-26: `oryd/kratos:v26.2.0` in docker on a small VPS, published as `auth.pidgeiot.com` through a Cloudflare Tunnel — config/bring-up live in the gitignored `.migration/`, secrets in the gitignored `secrets.env`), plus per-pigeon Ed25519 keypairs and compact binary bearer tokens (not JWTs) for device auth — see `dovecote` below. **Three hard-won Kratos production gotchas (task #47 cutover):** (0) the container MUST run `serve ... --watch-courier` — without that flag a single-instance Kratos silently QUEUES all outbound mail (`courier_messages` rows stuck at status 1, `send_count` 0, nothing ever reaches the SMTP provider, no error anywhere) — dev's docker-compose always had it; the first VPS bring-up didn't, which broke signup verification emails until the live launch test caught it; (1) the prod config MUST keep `cookies.domain: pidgeiot.com` — without it the session cookie is host-only to `auth.pidgeiot.com`, the browser never sends it to `api.pidgeiot.com`, dovecote's whoami 401s, and every dashboard list silently renders empty; (2) importing identities through the Kratos admin API mints NEW identity UUIDs — since `flocks.user_id` and each DO-resident `pigeon_acl` key on the original IDs, any future identity move must preserve IDs (see `.migration/remap2.sql` for the deferrable-FK in-place restore pattern; managed Postgres owner roles can't `DISABLE TRIGGER ALL`).

Full HTTP API reference (every dashboard + device route, auth models, request/response shapes): `docs/api.md`.

## Development commands

Three services run in parallel, each in its own terminal, from repo root unless noted:

```sh
# 1. Auth + DB (Kratos, Postgres, MailSlurper)
docker-compose -f infra/docker-compose.yml up --force-recreate

# 2. Edge backend (dovecote) — served at http://127.0.0.1:8787
cd dovecote && bunx wrangler dev --ip 127.0.0.1 --port 8787 --env dev

# 3. Frontend (fancier) — served at http://127.0.0.1:4455
cd fancier && dx serve --addr 127.0.0.1 --port 4455

# Live CSS rebuild while developing fancier
cd fancier && bunx @tailwindcss/cli -i ./assets/tailwind.css -o ./assets/styling/main.css --watch
```

- Kratos Admin UI: http://127.0.0.1:3000
- MailSlurper (local email capture): http://127.0.0.1:4436

Rebuilding the architecture diagram (fancier only):
```sh
cd fancier && bunx mmdc -i assets/architecture.mmd -o assets/images/architecture.svg -b transparent
```

Standard Cargo workflows apply per-crate (`cargo check -p dovecote`, `cargo check -p fancier`, `cargo check -p capsules`) — `dovecote` and `fancier` both target wasm/Workers, so a plain `cargo build` at the workspace root will not fully validate them; use `wrangler dev`/`dx serve` (above) or `worker-build`/`dx build` for real compilation checks.

Formatting: `tab_spaces = 2` (see `rustfmt.toml` in root, `dovecote/`, and `fancier/`) — this repo uses 2-space indentation everywhere, not the Rust default of 4.

## Architecture

### `dovecote` — edge router + Durable Objects

- `src/lib.rs` — the single Cloudflare Worker entrypoint (`#[event(fetch, ...)]`). Defines all HTTP routes on a `worker::Router`. Every route handler manually attaches CORS via `.with_cors(&cors)` and returns explicit error `Response`s (no `?`-propagation through route closures — see git history: "Unroll router let chains to avoid silent failures", "Response::Error() can't fail, unwrap instead of ?"). When adding routes, follow the existing pattern of `let Ok(x) = ... else { return Response::error(...) }` rather than `?`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [justins-engineering/pidgeiot](https://github.com/justins-engineering/pidgeiot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
