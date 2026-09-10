---
trigger: always_on
description: This repository contains a Node.js Telegram bot that retrieves Pixiv illustrations,
---

# Repository Guide

## Project Overview

This repository contains a Node.js Telegram bot that retrieves Pixiv illustrations,
manga, novels, rankings, authors, and ugoira animations and delivers them through
Telegram messages, inline queries, media groups, files, and Telegraph pages.

The project is an ES module package. Use the import aliases declared in
`package.json` (`#handlers/*`, `#handlers/pixiv/*`, `#handlers/telegram/*`,
`#handlers/utils/*`, `#config`, and `#db`) instead of introducing deep relative
imports.

## Architecture and Data Flow

- `app.js` is the main process. It loads and validates configuration, creates the
  Grammy bot, registers middleware and handlers, initializes PostgreSQL, checks
  and applies migrations, checks media tools, starts ranking updates and file
  cleanup, and handles graceful shutdown.
- `bot.js` constructs the Grammy bot and configures API throttling and automatic
  retries.
- `handlers/pixiv/` owns Pixiv requests, normalization, illustration caching and
  orchestration, URL generation, author and ranking retrieval, and ugoira
  conversion.
- `handlers/telegram/` owns input parsing, user settings, localization, formatting,
  keyboards, media groups, document and photo sending, and Telegraph publishing.
- `handlers/telegram/tg-sender.js` owns the state-oriented send workflow. Keep
  `app.js` limited to update routing and sender construction; do not move the
  workflow back into the bootstrap file.
- `handlers/telegram/settings-lifecycle.js` owns effective-setting resolution and
  authorized configuration commands. Pure precedence, flag, normalization, and
  sanitization rules live in `settings-resolver.js`; the `/s` grammar and flag
  registry live in `settings-command-parser.js`; Pixiv input parsing remains
  separate in `input-parser.js`.
- `handlers/telegram/input-parser.js` owns staged Pixiv/Phixiv URL and standalone
  ID extraction. Keep supported routes in its declarative matcher table rather
  than growing global replacement chains.
- `handlers/telegram/link-lifecycle.js` owns `/link` creation, management, and
  linked-message dispatch. Its PostgreSQL operations live in
  `chat-link-store.js`; do not return link state to generic settings updates.
- `handlers/common.js` contains shared download, concurrency, cache, logging, and
  memory-monitoring utilities.
- `db.js` is the PostgreSQL access layer. It exposes direct illustration APIs and
  MongoDB-shaped collection adapters still used by existing application code.
- `sql/schema.sql` is the complete schema for new installations.
  `sql/patches/` contains ordered incremental migrations.

The normal illustration path is: Telegram input -> Pixiv ID extraction -> cache or
PostgreSQL lookup -> Pixiv request -> normalization and URL construction ->
PostgreSQL update -> Telegram or Telegraph output.

## Setup and Commands

Use pnpm and a modern Node.js version that supports ES modules and package import
maps. The README documents Node.js greater than 15, but prefer a currently
supported Node.js release.

```bash
pnpm install
cp config_sample.js config.js
node initial.js
pnpm bot
pnpm test
```

- `node initial.js` creates the local media working directories.
- `pnpm bot` runs `app.js` with `WEBLESS=1` and is the reliable bot-only entry
  point.
- `pnpm all` runs `node app.js` and may attempt to load the currently missing
  `web.js` when web support is enabled.
- `pnpm web` currently targets a missing `web.js`; do not report it as working
  without restoring and validating that entry point.
- Production documentation uses PM2, but no PM2 ecosystem file, container file,
  CI workflow, or systemd unit is tracked in this repository.

Ugoira conversion requires `ffmpeg`, `mp4fpsmod`, and `unzip`. Startup checks for
these programs unless `DEPENDIONLESS=1` or development mode is enabled. Do not use
dependency-bypass modes as production validation.

## Configuration and Secrets

Create the ignored `config.js` from `config_sample.js`. Never commit or print real
PostgreSQL credentials, Pixiv cookies or CSRF values, Telegram bot tokens,
Telegraph tokens, salts, or private API endpoints.

Configuration may also be overridden by environment variables in
`handlers/utils/config-validator.js`. Relevant runtime flags include:

- `DBLESS=1`: use dummy database collections; this is not a production mode.
- `AUTO_APPLY_PATCHES=0`: refuse startup when automatic patches are pending.
- `WEBLESS=1`: do not start the optional web entry point.
- `TELEGRAM_API_SERVER`: use a custom Telegram Bot API root.
- `DEPENDIONLESS=1`: skip external media-tool checks.
- `dev=1`: enable development logging and skip dependency checks.

Verify configuration behavior from the current source before changing it. The
repository is in an incomplete PostgreSQL migration state:

- Runtime database initialization uses PostgreSQL, while validation still
  requires legacy MongoDB configuration.
- `config_sample.js` has no `web` object, although `app.js` reads
  `config.web.enabled` after startup.
- `config.js` is intentionally untracked. Application startup still requires
  it, while `db.js` receives the validated runtime configuration at database
  initialization so direct database tests do not need private credentials.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [my-telegram-bots/Pixiv_bot](https://github.com/my-telegram-bots/Pixiv_bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
