---
trigger: always_on
description: This is the single source of truth for AI assistants working on this repository. `CLAUDE.md` references this file.
---

# agent.md — AI Agent Guide for Squeeze Plex Hub

This is the single source of truth for AI assistants working on this repository. `CLAUDE.md` references this file.

---

## Project Overview

**Squeeze Plex Hub** is a full-stack Nuxt 4 application that acts as a protocol bridge between the Plex/Plexamp ecosystem and Squeezebox/LMS (Logitech Media Server) players. It:

1. Discovers LMS servers on the local network via mDNS.
2. Discovers Squeeze players connected to those servers.
3. Announces those players to Plex clients via GDM (Group Discovery Multicast) on UDP port 32412.
4. Translates Plex playback control commands into LMS/Squeeze RPC calls.
5. Publishes timeline state updates back to Plex clients.

**Version:** See `package.json` (`version` field)
**License:** MIT
**Author:** onmomo

---

## Stack

| Layer | Technology |
|-------|-----------|
| Meta-framework | Nuxt 4 (`nuxt` ^4.0.0) |
| Backend runtime | Nitro (H3 event handlers) |
| Frontend | Vue 3 Composition API |
| UI library | @nuxt/ui 4.1.0 |
| Language | TypeScript 5 (strict) |
| Package manager | Yarn 4.4.0 |
| Node version | v22 (see `.nvmrc`) |
| Testing | Vitest 3 + @nuxt/test-utils + happy-dom |
| Logging | Winston |
| XML | xml2js |
| LMS discovery | lms-discovery (mDNS) |
| LMS RPC | lms-squeeze-rpc-x |

---

## Directory Structure

```
squeeze-plex-hub/
├── app/                          # Vue 3 frontend (Nuxt app dir)
│   ├── components/
│   │   └── DiscoveredDevices.vue # Displays discovered LMS servers + players
│   └── pages/
│       └── index.vue             # Single-page entry point
├── server/                       # Nitro backend
│   ├── composables/              # Reusable server-side helpers
│   │   ├── useLogger.ts          # Winston logger factory
│   │   ├── usePlayerInfo.ts      # Retrieves player/server from DISCOVERY storage
│   │   ├── usePlayers.ts         # Fetches all players from storage
│   │   ├── useSqueezePlayer.ts   # Creates ExtendedSqueezePlayer instances
│   │   └── useXmlBuilder.ts      # Builds Plex-protocol XML responses
│   ├── lib/                      # Core business logic
│   │   ├── squeezePlexHub.ts     # Port resolution, Plex config constants
│   │   ├── squeezePlayer.ts      # ExtendedSqueezePlayer class
│   │   ├── plexApi.ts            # Plex server API communication
│   │   └── plexPlayerTimeline.ts # Timeline state management & interfaces
│   ├── middleware/
│   │   └── catchAll.ts           # Request logger middleware
│   ├── plugins/                  # Nitro startup plugins (run on server start)
│   │   ├── gdmAnnouncer.ts       # UDP multicast listener (port 32412)
│   │   ├── lmsScanner.ts         # LMS mDNS discovery & monitoring
│   │   └── timelinePublisher.ts  # Publishes player timeline every 1000ms
│   ├── routes/                   # H3 route handlers
│   │   ├── api/
│   │   │   └── players.get.ts    # GET /api/players
│   │   └── player/
│   │       ├── playback/         # Playback control endpoints
│   │       └── timeline/         # Timeline poll/subscribe endpoints
│   └── tasks/                    # Nitro scheduled tasks
│       ├── gdmDiscovery.ts       # Plex server GDM discovery (every minute)
│       ├── squeezePlayersScanner.ts # Squeeze player scan (every minute)
│       └── playQueueRefresher.ts # Refreshes queue on track end
├── public/                       # Static assets (logos, favicons)
├── .github/workflows/            # CI/CD (lint, build, test, Docker publish)
├── nuxt.config.ts                # Nuxt + Nitro configuration
├── vitest.config.ts              # Test runner configuration
├── eslint.config.mjs             # ESLint rules (Nuxt/TS/Vue)
├── .prettierrc                   # Prettier formatting rules
├── Dockerfile                    # Multi-stage Docker build
└── package.json                  # Scripts, deps, version
```

---

## Development Workflow

### Setup

```bash
# Install dependencies
yarn install

# Start dev server (binds 0.0.0.0 for Plex/LMS discovery)
yarn dev
```

### Common Commands

| Command | Purpose |
|---------|---------|
| `yarn dev` | Start development server |
| `yarn build` | Production build |
| `yarn start` | Run production build |
| `yarn lint` | Run ESLint checks |
| `yarn lint:fix` | Auto-fix ESLint issues |
| `yarn format` | Run Prettier formatting |
| `yarn test` | Run all tests (unit + integration) |
| `yarn test:coverage` | Run tests with coverage report |

### Running Tests

Tests are split into two Vitest projects:

- **unit** — `*.spec.ts` files (not `*.nuxt.spec.ts`), runs in Node environment
- **nuxt** — `*.nuxt.spec.ts` files, runs in Nuxt environment via `@nuxt/test-utils`

Test files are colocated with the source they test (e.g., `server/lib/squeezePlayer.spec.ts`).

```bash
yarn test               # run all tests
yarn test:coverage      # with LCOV coverage output
```

### Linting & Formatting

- ESLint extends `@nuxt/eslint-config` with TypeScript and Vue rules.
- Prettier: 140 char width, single quotes, no semicolons, no trailing commas.
- Always run `yarn lint:fix && yarn format` before committing.

---

## Code Conventions

### TypeScript

- Strict mode is enabled. Do not use `any` unless absolutely necessary and justified.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [onmomo/squeeze-plex-hub](https://github.com/onmomo/squeeze-plex-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
