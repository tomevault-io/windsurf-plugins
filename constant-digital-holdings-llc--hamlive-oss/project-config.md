---
trigger: always_on
description: Guidance for Claude Code and contributors working in this repo (`hamlive-oss`). This is the
---

# CLAUDE.md — Ham.Live (Open-Source Edition)

Guidance for Claude Code and contributors working in this repo (`hamlive-oss`). This is the
**public, MIT-licensed, self-hostable** edition of Ham.Live — a browser-first app for running
amateur-radio **nets** (coordinated on-air meetups where stations check in, exchange signal reports,
and follow a net-control-moderated agenda), with live presence, net discovery/following, and
real-time chat. It's the open-source sibling of the hosted Ham.Live service, shared for the
community to run and carry forward.

## Project status

Released **as-is** for the community to self-host. No guaranteed support, SLAs, or maintenance;
issues and PRs are handled best-effort; **forks are encouraged**. MIT — provided without warranty.
See `README.md` ("Project status") and `CONTRIBUTING.md`.

## Run it locally

```bash
npm install
npm run dev      # auto-creates .env, starts a local replica-set MongoDB, runs the app
```

- Open **http://localhost:3000**, enter any email. When email isn't configured, the **magic sign-in
  link is printed to the server console** (and shown on the page) — no paid accounts needed for a
  full local test drive.
- `npm run dev` (`scripts/dev.js`) starts MongoDB via **`mongodb-memory-server` as a single-node
  replica set** — required because the app uses change streams (see Subsystems). It's **in-memory
  and ephemeral** (data is lost on stop). For persistence, run `docker compose up -d` first.
- Other scripts: `npm start` (prod-style run), `npm run build` (tsc server + client), `npm run
  dev:watch` (tsc `-w` + nodemon), `npm run mongo:dev` (just the dev DB), `npm run gen-certs`
  (self-signed localhost HTTPS cert). Full, OS-specific setup is in **`INSTALL.md`**.

## Architecture & stack

- **Backend:** Node.js + Express + EJS (server-rendered views) + MongoDB (Mongoose).
- **Frontend:** TypeScript ES modules, native Custom Elements + reactive stores, **no bundler or
  framework**; Bootstrap 5; font **Outfit**.
- **Real-time:** MongoDB **change streams → Server-Sent Events** (with a polling fallback).
- **Auth:** magic-link email + optional Google OAuth; signed HTTP-only cookie sessions.
- **Chat:** GetStream (optional).
- **Optional external services** — all degrade gracefully when their keys are absent: SendGrid
  (email), GetStream (chat), QRZ (callsign lookup), Azure Maps (reverse-geocoding).
- Brand colors (in `client/dist/public/css/main.css`): `--hl-primary #dc8335` (orange),
  `--hl-secondary #6eb8c0` (teal), `--hl-light #f0eede` (cream). `--hl-success #3cce3c` is a utility
  green, **not** the brand color.

## Repo layout & the JS→TS migration ⚠️

```
client/   src/ (TS + SASS) → dist/ (compiled, served as static assets under dist/public/)
server/   src/ (TS — only for migrated modules) ; dist/ (what RUNS: controllers, models, routes, lib, views, bin)
docs/     technical docs (incl. docs/email-templates/ — reference SendGrid templates)
scripts/  setup.js, dev.js, devMongo.js
.env.example   every config option, documented
```

The project is **partway through a JavaScript → TypeScript migration**: some `server` modules have
TypeScript sources under `server/src/`; the rest are **maintained directly as JavaScript under
`server/dist/`**. **Before editing a `server/dist/**/*.js`, check for a matching
`server/src/**/*.ts`** — edit the `.ts` if it exists (it compiles to `dist`), otherwise edit the
`.js`. **EJS views live only in `server/dist/views/`** (no `src` counterpart). Subpath imports:
`#@server/*` → `server/dist/*`, `#@client/*` → `client/dist/public/js/*`. See `CONTRIBUTING.md`.

## Architectural direction — the `liveNet` pattern is THE standard ⭐ (mandatory for new work)

**This is the required architecture for the platform — not a suggestion, and not one option among
several.** Every new screen **MUST** follow it, and every screen still built the old way **MUST** be
ported to it whenever it is touched. The mandated destination is a **100% TypeScript, framework-free
MPA**.

> **`client/src/public/js/byView/liveNet/main.ts` is the reference implementation. Copy its shape for
> every new page. Do not invent a different structure, and do not introduce a front-end framework to
> "solve" what this pattern already solves.**

Ham.Live is a multi-page app: each screen is a server-rendered EJS view with a per-view TypeScript
entry point that hydrates it with reactive data + native Web Component widgets. Three layers, always:

1. **Typed endpoint → reactive store.** `lib/stores.ts` defines `ReactiveStore<T extends
   EndPointResponse>` (abstract); a concrete subclass per data domain (e.g. `LiveNetReactiveStore`,
   `FavoritesReactiveStore`) binds an `EndPointClient` and owns **all** data sync — a short-poll that
   **auto-upgrades to SSE** when the response carries an `ssePath`, hash-based change detection, an
   optimistic-update "in-flight window," and a subscriber pub/sub. **Widgets never fetch; the store
   does.**
2. **Native Web Component widgets.** `lib/widgets.ts` defines `HamLiveElement<T extends
   ReactiveStore>` (abstract), which **extends native `HTMLElement`** and implements
   `StoreSubscriber`. Widgets register as `hl-<tag>` custom elements (`customElements.define`), use a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Constant-Digital-Holdings-LLC/hamlive-oss](https://github.com/Constant-Digital-Holdings-LLC/hamlive-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
