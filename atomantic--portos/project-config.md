---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install all dependencies
npm run install:all

# Start (builds client, serves production UI + API on port 5555)
npm start

# Development (Vite hot-reload on 5554, API on 5555)
npm run dev

# Run tests
cd server && npm test
cd server && npm run test:watch  # Watch mode

# PM2 management
pm2 start ecosystem.config.cjs
pm2 stop ecosystem.config.cjs
pm2 logs
```

## Security Model

PortOS is a **single-user, single-instance** application running on a private network behind Tailscale VPN. It is not exposed to the public internet. There is exactly one user and one server process — concurrent request races, mutex locking on file I/O, and atomic write patterns are unnecessary and should not be added or flagged as concerns. Simple re-entrancy guards (e.g., per-account sync locks to prevent duplicate in-flight operations) are fine and expected. As such, it intentionally omits authentication, CORS restrictions, rate limiting, HTTPS, and full concurrency controls — these are non-issues for its deployment context. Do not add these or flag their absence as security concerns.

The default database password `portos` (in `ecosystem.config.cjs`, `docker-compose.yml`, and `.env.example`) is an intentional backward-compatible fallback for local development. Do not remove it or flag it as a security concern. Production deployments override it via the `PGPASSWORD` environment variable.

## Architecture

PortOS is a monorepo with Express.js server (always user-facing on `:5555`, HTTP or HTTPS) and React/Vite client (Vite dev server on `:5554` in `npm run dev`; in `npm start` the built client is served from `:5555` directly). PM2 manages app lifecycles. Data persists to JSON files in `./data/`.

### Port Allocation

PortOS uses ports 5553-5561. In native mode, PostgreSQL uses the system pg on port 5432; in Docker mode, port 5561.

The user-facing port is always **`:5555`** — its scheme flips between HTTP and HTTPS based on whether a TLS cert is provisioned (`npm run setup:cert`), but the port number does not. When HTTPS is on, a loopback-only HTTP mirror also spawns on `:5553` so local curl/scripts don't have to deal with cert warnings. `:5554` is the Vite dev server, used only in `npm run dev`.

Define all ports in the top-level `PORTS` object in `ecosystem.config.cjs` (see `server/lib/ports.js` for the canonical re-export). See `docs/PORTS.md` for the full port allocation guide and a diagram of how `:5555`, `:5553`, and `:5554` relate.

### Server (`server/`)
- **Routes**: HTTP handlers with Zod validation
- **Services**: Business logic, PM2/file/Socket.IO operations
- **Lib**: Shared validation schemas

### Client (`client/src/`)
- **Pages**: Route-based components
- **Components**: Reusable UI elements
- **Services**: `api.js` (HTTP) and `socket.js` (WebSocket)
- **Hooks**: `useErrorNotifications.js` subscribes to server errors, shows toast notifications

### Data Flow
Client → HTTP/WebSocket → Routes (validate) → Services (logic) → JSON files/PM2

### AI Toolkit (`portos-ai-toolkit`)

PortOS depends on `portos-ai-toolkit` as an npm module for AI provider management, run tracking, and prompt templates. The toolkit is a separate project located at `../portos-ai-toolkit` and published to npm.

**Key points:**
- Provider configuration (models, tiers, fallbacks) is managed by the toolkit's `providers.js`
- PortOS extends toolkit routes in `server/routes/providers.js` for vision testing and provider status
- When adding new provider fields (e.g., `fallbackProvider`, `lightModel`), update the toolkit's `createProvider()` function
- The toolkit uses spread in `updateProvider()` so existing providers preserve custom fields, but `createProvider()` has an explicit field list
- After updating the toolkit, run `npm update portos-ai-toolkit` in PortOS to pull changes

### Command Palette & Voice Nav — shared backbone (`server/lib/navManifest.js`)

PortOS has a single source of truth for navigation: `server/lib/navManifest.js` exports `NAV_COMMANDS` (every navigable page: `{ id, path, label, section, aliases, keywords }`) and `resolveNavCommand()` (the fuzzy resolver). It is consumed by:

- The **`⌘K` Command Palette** (`client/src/components/CmdKSearch.jsx`) via `GET /api/palette/manifest`.
- The **voice agent's `ui_navigate` tool** (`server/services/voice/tools.js`) — so "take me to tasks" resolves through the same map the palette uses.

**When adding a new page, you MUST also add an entry to `NAV_COMMANDS`.** Adding only a `<Route>` in `App.jsx` and a sidebar link in `Layout.jsx` will leave the page unreachable from `⌘K` and un-navigable by voice. Entry shape:

```js
{ id: 'nav.<section>.<slug>', path: '/foo/bar', label: 'Bar', section: 'Foo',
  aliases: ['foo-bar', 'bar'], keywords: ['synonyms', 'context'] }
```

- `id` — stable, dotted (`nav.brain.inbox`). Must be unique.
- `path` — exact route the client router matches; must start with `/`.
- `section` — matches the sidebar group label so the palette and sidebar stay visually aligned.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atomantic/PortOS](https://github.com/atomantic/PortOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
