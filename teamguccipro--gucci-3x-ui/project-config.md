---
trigger: always_on
description: Operational guide for AI agents working in this repo. Long-form human docs:
---

# CLAUDE.md

Operational guide for AI agents working in this repo. Long-form human docs:
`CONTRIBUTING.md` (setup, testing philosophy) and `frontend/README.md`.
Read those before large changes. This file is the short, must-follow version.
For a deep navigation map (request lifecycle, cron-job table, symptom → file
index, layering rules), read `docs/architecture.md` on demand — do not guess
file locations when it can answer in one hop.

## Stack
- Backend: Go 1.27 (`module github.com/mhsanaei/3x-ui/v3`), Gin, GORM.
  Runs Xray-core as a managed child process (`internal/xray/process.go`) and
  imports `github.com/xtls/xray-core` for config types + gRPC stats/handler/router
  API. MTProto inbounds run a second managed child — the `mtg-multi` binary
  (a multi-secret mtg fork — NOT a Go dependency; its prebuilt release binary is
  fetched at image/release build time by `DockerInit.sh` + `release.yml`,
  panel-side code in `internal/mtproto/`) — outside Xray, one process per inbound
  serving each
  client's FakeTLS secret via the fork's `[secrets]` section (plus per-client
  ad-tags via `[secret-ad-tags]` and per-client data quota / expiry via
  `[secret-limits]`, mapped from the client's `totalGB`/`expiryTime`). Client,
  ad-tag and quota/expiry edits are hot-applied through the fork's management API
  (`PUT /secrets`, bearer-token guarded) so connections survive; the manager
  falls back to a process restart on older binaries. A client's panel-side
  traffic reset also calls `POST /secrets/{name}/reset-quota` so a renewed client
  is not re-blocked by the sidecar's quota counter.
- Storage: SQLite by default (`/etc/x-ui/x-ui.db` on Linux; the executable dir on
  Windows), PostgreSQL optional (`XUI_DB_TYPE` / `XUI_DB_DSN`). The CGo SQLite
  driver (`mattn/go-sqlite3`) needs a C compiler — `CGO_ENABLED=0` builds fail.
- Frontend: React 19 + Ant Design 6 + Vite 8 + TypeScript in `frontend/`,
  built into `internal/web/dist/` (gitignored) and embedded via `embed.FS`.

## Repo map
- `main.go` — entry point + `x-ui` CLI (run, migrate, migrate-db, setting, cert).
- `internal/config/` — env parsing (XUI_DEBUG, XUI_LOG_LEVEL, XUI_LOG_FOLDER,
  XUI_BIN_FOLDER, XUI_SKIP_HSTS, XUI_PORT, XUI_DB_*).
- `internal/database/` + `internal/database/model/` — GORM schema (~24 models;
  Inbound, Client, Setting, User are the core), inbound Protocol enum,
  AutoMigrate + hand-written migrations in `db.go`.
- `internal/xray/` — Xray child-process lifecycle, config generation, gRPC API.
- `internal/xray/geodata/` — streaming geosite/geoip `.dat` reader (cached
  category index + paged entries) and `geosite:`/`geoip:`/`ext:` token parsing.
- `internal/mtproto/` — MTProto inbounds via the bundled `mtg-multi` binary.
- `internal/amneziawg/` — AmneziaWG protocol shape: instance/peer derivation
  from an inbound, 3.1 obfuscation param generation + validation, port-forward
  spec parsing.
- `internal/amneziawgnet/` — embedded AmneziaWG runtime: amneziawg-go device
  over a gVisor userspace netstack, per-inbound reconcile manager, TCP/UDP
  relay into a loopback per-peer-auth SOCKS5 Xray inbound, port-forward
  listeners, per-peer IPv6 egress aliases.
- `internal/pia/` — PIA WireGuard protocol client (auth, signed server list, `/addKey`).
- `internal/sub/` — subscription server (raw / JSON / Clash).
- `internal/eventbus/` — in-process pub/sub (outbound/node health, xray.crash,
  cpu.high, memory.high, login.attempt).
- `internal/logger/`, `internal/util/` (link, crypto, sys, ldap, …),
  `internal/tunnelmonitor/` — shared infrastructure.
- `internal/web/` — Gin server (embeds `dist/` + `translation/`).
  - `controller/` — panel + REST API handlers; OpenAPI at /panel/api/openapi.json.
  - `service/` — business logic (InboundService, SettingService, XrayService,
    node sync); subpackages tgbot/, email/, outbound/, panel/, integration/.
  - `job/` — 18 cron jobs (traffic, fail2ban IP-limit, node heartbeat/sync, LDAP,
    CPU/memory watchdogs, …); full table in `docs/architecture.md` §5.4.
  - `middleware/`, `entity/`, `global/`, `session/` (CSRF), `network/`,
    `runtime/` (master/sub-node over mTLS), `websocket/`.
  - `locale/` + `translation/` — i18n, 13 embedded locale JSON files.
- `frontend/` — React + TS source (see `frontend/CLAUDE.md`).
- `tools/openapigen/` — Go generator that emits frontend types + Zod/JSON schemas
  into `frontend/src/generated/` from Go structs. The OpenAPI doc itself
  (`frontend/public/openapi.json`) is assembled from those + `endpoints.ts` by
  `frontend/scripts/build-openapi.mjs`.
- `docs/` — separate Next.js/Fumadocs site (pnpm, own CI in `docs-ci.yml`,
  outside `make verify`). Holds a THIRD independent implementation of
  link/subscription generation in `docs/lib/xray/` — check it whenever
  share-link or install-command output changes.

## Hard rules (non-negotiable)
- Fix size must match bug size. Find the root cause, then make the SMALLEST
  change that removes it — a one-line guard beats a new subsystem. A small bug
  does not earn new columns, jobs, abstractions, config knobs or helper layers.
  If a fix genuinely needs new architecture, say so and get agreement first;
  never ship it unasked next to the fix.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TEAMGUCCIPRO/GUCCI-3X-UI](https://github.com/TEAMGUCCIPRO/GUCCI-3X-UI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
