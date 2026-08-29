---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Irminsul is a full-stack Minecraft authentication server implementing the Yggdrasil protocol (compatible with authlib-injector). It features a web UI for account management, session-based auth, and skin/cape texture hosting.

## Tech Stack

- **Runtime:** Bun (required)
- **Framework:** Nuxt 4 (Vue 3 SSR with Nitro server engine)
- **Frontend:** Tailwind CSS v4 + DaisyUI v5
- **Backend:** Nitro server routes + server utils
- **Database:** MongoDB (user data, tokens) + Redis (sessions)
- **Validation:** Zod
- **Testing:** Vitest + @nuxt/test-utils
- **Language:** TypeScript (strict mode, ES2022 target)

## Commands

```bash
bun run dev        # Start dev server with HMR
bun run build      # Production build
bun run preview    # Preview production build
bun run prod       # Build then run production server
bun run lint       # Lint with oxlint (type-aware)
bun run lint:fix   # Lint and auto-fix
bun run fmt        # Format with oxfmt
bun run fmt:check  # Check formatting
bun run test       # Run all tests with Vitest
bun run test -- tests/utils/ban.repository.test.ts  # Run a single test file
```

### CLI Tool (`cli/`)

Standalone Bun CLI for setup and migration (separate package in `cli/`):

```bash
bun cli/src/init.ts      # Run CLI (dev)
cd cli && bun run build  # Build CLI
```

Two modes: **fresh install** (configures MongoDB/Redis, generates `.env`) and **GHAuth migration** (imports users, skins, config from legacy GHAuth).

## Architecture

### Nuxt Directory Structure

- `app/` — Client-side code (Nuxt 4 app directory)
  - `pages/` — File-based routing (each `*.vue` maps to a URL)
  - `components/` — Auto-imported Vue components
  - `composables/` — Auto-imported composables (e.g., `useUser()`, `useSettings()`)
  - `layouts/` — Layout components (`default.vue`)
  - `stores/` — Pinia stores
  - `plugins/` — Client plugins
  - `assets/css/` — Tailwind CSS and theme definitions
- `server/` — Nitro server-side code
  - `api/` — API routes (file-based: `server/api/auth/login.post.ts` → `POST /api/auth/login`)
  - `routes/` — Non-API routes (e.g., Yggdrasil protocol endpoints)
  - `plugins/` — Nitro server plugins (DB init, logging, key generation)
  - `middleware/` — Server middleware (session resolution)
  - `utils/` — Auto-imported server utilities (DB repos, auth helpers, crypto)
- `tests/` — Vitest test files (`tests/server/`, `tests/utils/`)
- `cli/` — Standalone Bun CLI for setup/migration (separate package)

### Server Routes & API

Nitro file-based API routes. Convention:

- `server/api/**/*.get.ts` — GET endpoints
- `server/api/**/*.post.ts` — POST endpoints
- `server/routes/yggdrasil/` — Yggdrasil protocol endpoints

All server utils in `server/utils/` are auto-imported in server context.

### Data Fetching

- **Page data:** Use `useAsyncData` + `$fetch('/api/...')` in pages/components
- **User interactions:** Use `$fetch` directly for mutations (form submits, button clicks)

### Settings Table (`server/utils/settings.repository.ts`)

MongoDB collection `settings` stores system configuration (SMTP, auth policies, etc.).

**Schema:** `{ key: string, value: any, source: string }`

- `key` — must follow `"category.name"` format (e.g. `"smtp.host"`, `"auth.requireEmailVerification"`)
- `value` — any MongoDB-supported type
- `source` — origin of the setting; all built-in settings use `"irminsul.builtin"`
- Unique index on `key`

**Built-in settings** (initialized on startup, won't overwrite existing values):

| Key                             | Default | Description                                              |
| ------------------------------- | ------- | -------------------------------------------------------- |
| `smtp.host`                     | `""`    | SMTP server host                                         |
| `smtp.port`                     | `465`   | SMTP port                                                |
| `smtp.secure`                   | `true`  | Use TLS                                                  |
| `smtp.user`                     | `""`    | SMTP username                                            |
| `smtp.pass`                     | `""`    | SMTP password                                            |
| `smtp.from`                     | `""`    | Sender address (e.g. `"Irminsul <noreply@example.com>"`) |
| `auth.requireEmailVerification` | `false` | Whether email verification is required                   |
| `general.announcement`          | `""`    | Homepage announcement text (empty = title only)          |

**API:** `getSetting(key)`, `setSetting(key, value, source)`, `getSettingsByCategory(category)`, `getSettingsMap(keys)`, `deleteSetting(key)`

### Authentication Flow

1. Login form submits email + password + Altcha proof-of-work via `$fetch('/api/auth/login', { method: 'POST' })`
2. Server looks up user in MongoDB, verifies password (Argon2id or legacy, auto-upgrades)
3. Yggdrasil token pair created (accessToken + clientToken), stored in user document
4. Redis session created, `irmin_session` cookie set
5. Server middleware resolves session → `event.context.user`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daidr/irminsul](https://github.com/daidr/irminsul) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
