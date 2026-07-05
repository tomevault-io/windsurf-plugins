---
trigger: always_on
description: Guidance for AI coding agents (and the humans driving them) contributing to
---

# AGENTS.md

Guidance for AI coding agents (and the humans driving them) contributing to
Lurker. If you're a person, [`README.md`](README.md) is the friendlier intro;
this file is the fast, dense orientation an agent needs to make a correct change
and open a clean PR. Everything here is enforced by CI or by review — following
it is the difference between a merge and a round of change requests.

Lurker is a self-hosted IRC client: an always-on Node server that stays
connected to IRC and keeps full history, plus a Vue 3 web UI that reattaches
from any browser. License is **MPL-2.0** throughout.

Please note that while Lurker is an LLM/Agent friendly project, the project is
under the direction of Brad Root (amiantos), and just because a feature
can be added, does not mean it will be accepted. Lurker is deliberately lo-fi
in many ways, despite the feature set being very modern. So, before you
commit yourself to working on something that excites you in Lurker, be sure to
have a discussion with amiantos in the #lurker IRC channel on Libera.Chat to
make sure it fits the project ethos.

It should also be assumed most GitHub issues are 'assigned' by default to amiantos.
If you wish to work on an issue, be sure to ask him about it in the channel first,
to avoid potentially wasted time. Also note that while I have been saying 'you'
in the last two paragraphs, but you should probably let your operator handle the
talking, unless they really would want you to sign onto IRC to try to talk to amiantos
directly. Please let him know you're an agent up-front just to be nice.

## Repository layout

```
server/            TypeScript on Node, run via tsx (no build step)
  server.ts        Process lifecycle: HTTP server, WS hub, IRC manager, identd
  app.ts           Express app construction + route wiring (edition-gated)
  routes/          One Express router per resource (auth, networks, uploads…)
  services/        IRC manager + connection, WS hub, image pipeline, identd,
                   MCP server, connect scheduler, upload providers
  services/verbs/  Shared ACTION registry (sendMessage, searchMessages, …)
                   consumed by BOTH the WS command path and the MCP server
  db/              better-sqlite3 data access, one module per table
  db/index.ts      The single DB connection + schema migrations
  middleware/      auth (session cookie), apiAuth (bearer token), nodeAuth
  utils/           small helpers (edition, ident, secretCrypto, username…)
  types/           ambient *.d.ts shims for untyped deps
  test-utils/      testApp.ts — the integration-test harness
shared/            Code imported by BOTH server and client
  settingsRegistry.ts  Single source of truth for user settings
vue_client/        Vue 3 + Vite + Pinia + vue-router SPA
  src/stores/      Pinia stores, one per domain (buffers, auth, settings…)
  src/components/  Vue components (+ settings-panes/)
  src/composables/ useSocket, usePresence, useKeyboardShortcuts, …
  src/views/       Login, Chat (Desktop/Mobile), Settings, InviteAccept
  src/lib/         framework-free logic (e.g. virtualBuffers)
docs/              SELF_HOSTING, digitalocean, MCP, DESIGN_TOKENS
deploy/            operator deploy scripts (not needed for app dev)
integrations/      autonotes
```

Tests live **next to the code** they cover as `*.test.ts`.

## Setup & dev

- **Node 22** — matches CI and the Docker runtime. better-sqlite3 + sharp are
  native; stay on 22 to avoid ABI surprises.
- `npm run install:all` — installs root **and** `vue_client/` deps.
- `cp .env.example .env` — defaults are documented inline.
- `npm run dev` — runs server + client concurrently.
  - ⚠️ The server **auto-connects (by default) on boot to whatever IRC networks are
    in its database**. Don't point a dev instance at networks you don't control, and
    don't assume "the server is running" is a safe way to test — prefer the
    typecheck/lint/test gate below, which needs no live IRC.

## The CI gate — run before every PR

CI ([`.github/workflows/test.yml`](.github/workflows/test.yml)) runs these on
every PR to `main`, and **all must pass**:

| Command                    | What it is                         |
| -------------------------- | ---------------------------------- |
| `npm run typecheck`        | server type-check (`tsc`, no emit) |
| `npm run typecheck:client` | client type-check (`vue-tsc`)      |
| `npm run lint`             | **oxlint**                         |
| `npm run format:check`     | **oxfmt**                          |
| `npm test`                 | **Vitest**                         |

`npm run check` bundles the first four. Run it plus `npm test` and you've
reproduced the gate locally.

## Code style & conventions

- **Formatter is `oxfmt`, NOT Prettier.** Run `npm run format`. Do **not** run
  Prettier or ESLint — they fight oxfmt's house style (single quotes, 100-col
  width) and will make `format:check` fail. Linter is `oxlint`.
- **ESM with `.js` import specifiers.** The project is `"type": "module"` with
  `verbatimModuleSyntax`. Import sibling TypeScript files using a `.js`
  extension even though the file on disk is `.ts`:
  ```ts
  import { createUser } from '../db/users.js'; // file is users.ts
  ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amiantos/lurker](https://github.com/amiantos/lurker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
