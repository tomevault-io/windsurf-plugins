---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

PendingDNS is a lightweight, API-driven **authoritative** DNS server (UDP + TCP), bundled with a REST API for managing zone records, a public HTTP/HTTPS redirect/proxy server, automatic Let's Encrypt certificate generation, and periodic health checks. All state lives in Redis.

## Commands

```bash
npm start          # run the whole app (node server.js)
npm test           # run the test suite (requires a local Redis on 127.0.0.1:6379)
npm run lint       # ESLint 9 (flat config: eslint.config.js)
npm run licenses   # regenerate licenses.txt (license-report)
```

Running the app or tests requires a reachable Redis and a config with a valid `acme.email` (the bootstrap in `server.js` exits otherwise). For local runs, either set it in `config/development.toml` or pass `--acme.email=you@example.com`.

### Tests

- Built on the Node.js built-in test runner (`node:test`). `npm test` runs `NODE_ENV=test node --test --test-force-exit --test-concurrency=1 test/*.test.js`.
- `NODE_ENV=test` loads `config/test.toml`, which points Redis at a **dedicated database (db 15)**. `test/helpers.js#flushTestDb` wipes db 15 between cases and **refuses to run unless the Redis URL ends in `/15`** — so tests never touch dev (db 2) or prod data. Always run tests with `NODE_ENV=test`.
- `--test-concurrency=1` is required because all test files share db 15. `--test-force-exit` is required because requiring `lib/db` opens persistent ioredis connections; every test file must also call `closeDb()` in an `after()` hook or the process hangs.
- Some `cached-resolver`/`dns-server` tests do real DNS lookups (e.g. `one.one.one.one`); they are written to skip or tolerate missing network.

Run a single file or filter by test name:

```bash
NODE_ENV=test node --test --test-force-exit test/zone-store.test.js
NODE_ENV=test node --test --test-force-exit --test-name-pattern="wildcard" test/*.test.js
```

### Building standalone executables

Executables are built with `@yao-pkg/pkg` (the maintained `pkg` fork) via the `pkg` block in `package.json`, targeting node24. The org-wide `../emailengine-api/upload.sh <repo>` script installs `@yao-pkg/pkg` globally, then runs `npm run build-source` followed by `npm run build-dist` (or `build-dist-fast` for an unsigned test build) and signs/notarizes/uploads the artifacts. Output binaries are named `pending-dns-<target>` under `ee-dist/`.

Because workers are loaded through **dynamic `require()` / `worker_threads`** (see below), `pkg.scripts` must list `lib/**/*.js` and `workers/**/*.js` or the worker code is missing from the snapshot. Runtime files read via `__dirname` (`lib/lua/health.lua`, `config/*.pem`) must be in `pkg.assets`. Do **not** add `pkg.options` like `max_old_space_size`: the workers forward `argv`, and an injected V8 flag leaks in as a bogus module path and crashes every worker.

## Architecture

### Process model (the most important thing to understand)

`server.js` is the supervisor. For each enabled subsystem (`api`, `dns`, `public`, `health`) it spawns a **worker thread** running `workers/<type>.js`, and restarts it on exit. Each `workers/<type>.js` is itself a small bootstrap that:

1. installs crash handlers via `lib/close-process.js#installCrashHandlers` (shared with `server.js`) plus optional Sentry error reporting via `lib/sentry.js#initSentry` (only when `SENTRY_DSN` / `[sentry] dsn` is set). **`closeProcess` owns the exit unconditionally**; when a reporter is active it waits out a short flush window (`FLUSH_GRACE`) first, and Sentry's integrations only capture + flush. Nothing may delegate the exit to Sentry: `onUncaughtExceptionIntegration` returns early in a worker thread and stands down whenever another `uncaughtException` listener exists (`lib/logger.js` registers one), so a delegated exit never happens and the worker is left running with `closing` latched - deaf to SIGTERM and to every later crash,
2. uses Node `cluster` to fork `config[type].workers` processes (or runs in-thread when `workers === 1` and no user/group drop is configured),
3. dynamically `require`s the implementation: ``require(`../lib/${type}-server.js`)`` (or `-worker.js` for `health`) and calls it,
4. drops privileges via `config.process.user`/`group` after ports are bound.

So `lib/<type>-server.js` are entry points reached **only through dynamic requires** — static analysis (and `pkg`) won't find them. The implementations are: `lib/api-server.js`, `lib/dns-server.js`, `lib/public-server.js`, `lib/health-worker.js`.

### Configuration (`wild-config`)

Config is loaded by `wild-config` from `config/default.toml` merged with `config/<NODE_ENV>.toml`, then overridden by CLI args (`--dns.port=53`) and `appconf_*` env vars. `NODE_CONFIG_PATH` points to an external file in production (see `systemd/pending-dns.service`). Values are coerced to the type of the default (a string env var for a numeric default becomes a number). Config is read from `process.cwd()/config`, **not** from `__dirname` — relevant for packaged binaries, which need a `config/` directory next to them.

### Redis data model (`lib/zone-store.js`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [postalsys/pending-dns](https://github.com/postalsys/pending-dns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
