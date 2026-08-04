---
trigger: always_on
description: Alita Robot is a Telegram group-management bot written in **Go 1.26** on top of
---

# Repository Guidelines

Alita Robot is a Telegram group-management bot written in **Go 1.26** on top of
the **gotgbot/v2** library (`v2.0.0-rc.35`). It provides admin tools, filters,
notes, greetings, anti-flood / anti-raid / anti-spam, captcha verification,
warns, locks, backups, connections, reactions and multi-language support
(en, es, fr, hi, ru, pt, id).

> `CLAUDE.md` and `GEMINI.md` are symlinks to this file — **AGENTS.md is the single
> source of truth** for agent/contributor guidance. Edit this file only.

## Maintaining This File

This file is **not** auto-generated. When you make changes that affect anything
documented here — build pipeline, scripts, env vars, routes, key systems,
dependencies, directory layout, or code-style rules — update the relevant
section in the same change so it stays accurate. `CLAUDE.md` and `GEMINI.md` are
symlinks to this file, so edit `AGENTS.md`.

---

## 0. Maintaining this document (READ FIRST)

**This file is a living knowledge base. Keep it current as you work.**

- When you discover something **non-obvious, load-bearing, or surprising** about
  the codebase (a hidden coupling, a "why it's done this way" decision, a footgun,
  a corrected fact, a new subsystem), **record it here in the most relevant
  section before you finish the task.** Treat it as part of "done."
- **Consolidate, don't append.** Before adding a note, find where it belongs and
  merge it with what's there. Fix stale/contradictory statements in place rather
  than stacking a second version next to them. Prefer one accurate sentence over
  three vague ones. Remove notes that have become false.
- **Be specific and verifiable**: name the file/function/env-var/table/constant.
  A future agent must be able to act on the note without re-deriving it.
- **Verify before trusting.** This document reflects the code at the time each
  note was written. If a note names a file, function, flag, table, or default,
  confirm it still exists before relying on it — and if it has changed, update the
  note as part of your change.
- Don't duplicate what the code/tests/git history already make obvious; capture
  the *why* and the *gotcha*, not a restatement of the code.
- This document was last fully reconciled against the source by a whole-codebase
  read; sections below marked with ⚠️ call out where older docs had drifted.

---

## 1. Mental model — how it fits together

A Telegram **update** flows like this:

```
Telegram ──► (polling updater  OR  webhook /webhook POST)
          ──► ext.Dispatcher (tracing.TracingProcessor wraps each update in a span;
                              dispatcherErrorHandler classifies errors → Noop)
          ──► handlers, executed in HANDLER-GROUP order (negative → 0 → positive)
                 • group -10/-5/-2/-1 : early interceptors (captcha pending, antiraid,
                                        antispam, passive Users tracker)
                 • group 0            : normal command handlers (return ext.EndGroups)
                 • group 4..10        : message watchers (antiflood, locks, blacklists,
                                        filters, reactions, reports) (return ext.ContinueGroups)
          ──► handler reads/writes DB (GORM/Postgres) through per-domain repos,
              which read-through a Redis cache; replies via i18n + media/formatting
```

Big architectural facts an agent must hold in mind:

- **Config and the DB connection are opened in package `init()` functions, not in
  `main()`.** Importing `alita/config` loads+validates config into the global
  `config.AppConfig`; importing `alita/db` opens the Postgres connection. Both
  short-circuit for CLI flags (`--version`/`--health`) and when their required env
  is unset (so tests can import them). Do **not** move this into `main()`.
- **The DB layer is split into per-domain sub-packages** (`alita/db/<domain>/`)
  with all GORM structs in `alita/db/models/`. `alita/db/db.go` is a
  backward-compat shim that re-exports model types (`db.User = models.User`), cache
  helpers, and TTL constants. ⚠️ Older docs described a flat `alita/db/*_db.go`
  layout — that no longer exists.
- **Schema source of truth is raw SQL in `migrations/*.sql`**, applied by a custom
  runtime engine (`alita/db/migrations/runner.go`), **not** `gorm.AutoMigrate`.
  GORM struct tags only affect runtime CRUD. Tests bootstrap schema via SQLite
  `AutoMigrate` (`testmain_test.go`), so struct↔SQL drift is possible and not
  caught by tests — keep them in sync manually.
- **Cache is Redis-only** (via `eko/gocache` + `go-redis`). There is no in-memory
  production fallback. Every cache helper is nil-safe: when the marshaler is nil
  it bypasses caching and hits the DB directly.
- **Modules self-register in `init()`** and load in ascending-priority order; the
  Help module loads last (deferred) so it can render every module's metadata.
- **Callback data uses a versioned codec** (`<namespace>|v1|<url-encoded>`) capped
  at Telegram's 64-byte limit — never `strings.Split` raw callback data.

---

## 2. Project structure

- **`main.go`** — process entry point (CLI flags, bootstrap, polling/webhook
  branch, dispatcher, shutdown wiring, custom Bot-API rewrite transport).
- **`alita/`** — application code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Divkix/Alita_Robot](https://github.com/Divkix/Alita_Robot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
