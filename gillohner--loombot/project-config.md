---
trigger: always_on
description: A Deno-based Telegram bot framework with sandboxed service execution and Pubky decentralized storage
---

# loombot — Telegram community bot

A Deno-based Telegram bot framework with sandboxed service execution and Pubky decentralized storage
integration. Services are isolated in zero-permission Deno subprocesses and communicate via
stdin/stdout JSON.

## Quick Reference

- **Runtime:** Deno (not Node.js)
- **Framework:** grammY (Telegram bot library)
- **Database:** SQLite (via deno.land/x/sqlite)
- **Language:** TypeScript (strict)
- **Formatting:** Tabs, 100 char line width (`deno fmt`)
- **Linting:** `deno lint` (recommended rules)
- **Testing:** `deno test`
- **Dev:** `deno task dev` (polling mode with --watch)
- **Prod:** `deno task serve` (webhook mode)
- **Fresh start:** Snapshots auto-clear on every process start. Edit `config.yaml` and restart the
  bot to pick up changes. Deleting `bot.sqlite` is rarely needed.

## Architecture

```
Telegram → grammY Bot → Router Middleware → Dispatcher
                                                ↓
                                          Snapshot (routing table)
                                                ↓
                                          Sandbox Host (Deno subprocess, zero permissions)
                                                ↓
                                          Service source file (resolves @sdk/ via import map)
                                                ↓
                                          ServiceResponse → Telegram Adapter → User
```

### Core Flow

1. Telegram update arrives (polling or webhook)
2. Router handles admin commands (`/start`, `/config`) or dispatches to services
3. Dispatcher loads routing snapshot for the chat, finds matching service route
4. Sandbox host spawns a Deno subprocess pointed at the service source file; the subprocess resolves
   `@sdk/` / `@eventky/` / `npm:` imports via the project's `deno.json` import map
5. Service returns `ServiceResponse` via stdout JSON
6. Response adapter converts to Telegram API calls

Per-chat configuration happens via the inline `/config` menu (admin-only), which writes feature
toggles and overrides into the `chat_feature_overrides` table. The operator-level defaults live in
`config.yaml` and are loaded once at startup.

## Project Structure

```
src/
├── main.ts                        # Entry point (polling vs webhook)
├── bot.ts                         # Bot init, middleware composition
├── core/
│   ├── config.ts                  # Process-wide env flags (NODE_ENV, DEBUG, …)
│   ├── config/loader.ts           # Parse + validate config.yaml at startup
│   ├── config/schema.ts           # Zod schema for config.yaml
│   ├── config/runtime.ts          # Loaded operator-config singleton + hash
│   ├── config/merge.ts            # Resolve per-chat effective feature list (operator + overrides)
│   ├── config/store.ts            # SQLite persistence (overrides, snapshots, writes, pins)
│   ├── config/migrations.ts       # DB schema migrations
│   ├── dispatch/dispatcher.ts     # Event routing → sandbox execution → state mgmt
│   ├── sandbox/host.ts            # Deno subprocess with zero permissions
│   ├── snapshot/snapshot.ts       # Effective config → routing table, caching
│   ├── scheduler/scheduler.ts     # Periodic meetups broadcast loop
│   ├── scheduler/pin_store.ts     # SQLite-backed pin + last-fired tracking
│   ├── state/state.ts             # In-memory state (chatId+userId+serviceId keyed)
│   ├── pubky/pubky.ts             # Re-export shim for calendar_meta helpers
│   ├── pubky/calendar_meta.ts     # Fetches calendar metadata from Pubky (used by /config)
│   ├── pubky/writer.ts            # Admin-approval write queue
│   ├── pubky/writer_store.ts      # Writer SQLite persistence
│   ├── ttl/store.ts               # Message auto-deletion scheduling
│   └── util/
│       ├── logger.ts              # Structured JSON logging
│       └── utils.ts               # Misc helpers (command normalization, callback data parsing)
├── middleware/
│   ├── router.ts                  # Command routing, admin commands
│   ├── response.ts                # ServiceResponse → Telegram API
│   ├── admin.ts                   # Permission checks
│   └── config_ui/                 # /config inline menu: features, calendars, welcome, periodic
├── services/
│   └── registry.ts                # Operator-shipped service registry
├── adapters/
│   └── telegram/
│       ├── adapter.ts             # Telegram API integration
│       └── ui_converter.ts        # UI abstraction → Telegram format
└── types/
    ├── routing.ts                 # RoutingSnapshot, CommandRoute, ListenerRoute
    ├── sandbox.ts                 # ExecutePayload, sandbox types
    └── services.ts                # Service protocol types

packages/
├── sdk/                       # Service SDK (imported by every service via the @sdk/ import map)
│   ├── mod.ts                 # Public API surface
│   ├── service.ts             # defineService() + ServiceDefinition
│   ├── events.ts              # CommandEvent, CallbackEvent, MessageEvent
│   ├── state.ts               # state.replace/merge/clear()
│   ├── responses/
│   │   ├── types.ts           # ServiceResponse union type
│   │   ├── factory.ts         # reply(), edit(), photo(), pubkyWrite(), etc.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gillohner/loombot](https://github.com/gillohner/loombot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
