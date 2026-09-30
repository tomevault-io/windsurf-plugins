---
trigger: always_on
description: Bun + TypeScript server that converts a streaming-service link into links on other platforms. SSR pages via nano-jsx, fragment updates via htmx, small Stimulus controllers for client behavior.
---

# IDHS (I Don't Have Spotify) — agent instructions

Bun + TypeScript server that converts a streaming-service link into links on other platforms. SSR pages via nano-jsx, fragment updates via htmx, small Stimulus controllers for client behavior.

## Directory map

- `src/index.ts` — HTTP server (`Bun.serve` with `routes`): `GET /`, `POST /search` (htmx HTML fragment), `POST /api/search`, `POST /api/auth/request-code`, `POST /api/auth/verify-code`, `GET /api/status`, `GET /verify` (Cloudflare challenge landing that redirects back), static fallback for `public/`.
- `src/parsers/` — identify the incoming link's platform, extract normalized metadata + search query.
- `src/adapters/` — turn the query into outbound links per destination platform.
- `src/abuse/` — public instance gate: stateless OTP (`otp.ts`), email policy (`email.ts`), Plunk client (`plunk.ts`), session tokens (`session.ts`, cookie-only, no bearers), per-email quota (`quota.ts`), Durable Object (`quota-do.ts`), gate checks + auth route handlers (`gate.ts`, `routes.ts`).
- `src/services/` — `search.ts` orchestration, in-memory `cache.ts`, `metadata.ts` helpers, `musicbrainz.ts` fallback (fills missed adapters from curated streaming relations).
- `src/schemas/` — zod route schemas (`auth.schema.ts` covers the gate endpoints); `src/config/` — enums, constants, env.
- `src/utils/` — `http-client.ts` (all outbound HTTP goes through `impit` here), logger, scraper, service guard (per-service upstream budgets + circuit breaker).
- `src/views/` — nano-jsx components/layouts/pages (`components/gate.tsx` is the inline OTP panel), `controllers/*.js` (Stimulus, plain JS), `css/index.css` (Tailwind).
- `tests/` — integration tests (`api`, `page`, `abuse/gate`), unit tests (`parsers`, `search`, `utils`, `abuse`, `spotify-totp`); `tests/utils/http-mock.ts` stubs `HttpClient` statics (non-2xx mocks carry a body snippet, mirroring production `HttpClientError.body`); `tests/mocks/` holds HTML/JSON snapshots.
- `scripts/fetch-snapshots.ts` — regenerates `tests/mocks/` from live URLs (network).
- `build.config.ts` — Tailwind CLI + browser JS bundling for `public/assets/`.
- `API.md` — public API contract (includes the gate auth endpoints; cookie session, no bearer flow — programmatic clients target self-host); `README.md` — user/operator narrative.

## Prerequisites

- Bun 1.4.2 is the locally installed runtime (`bun --version`). Note the toolchain floats: Dockerfile builds on `oven/bun:1-alpine` and CI uses `oven-sh/setup-bun@v1` with no version input, so CI may run a newer Bun than local.
- `bun install` before anything else (Dockerfile uses `--frozen-lockfile`). Never read `.env` values; the variable *names* are listed in `.env.test`, and README links the portals that issue the real values.

## Commands (run from repo root)

| Command | What it does | Status |
|---|---|---|
| `bun run dev` | asset watcher + `bun run --watch src/index.ts` (port `PORT` or 3000) | defined, not run here |
| `bun run build` | production CSS + browser JS into `public/assets/` | defined, not run here |
| `bun run build:prod` | `--compile` single binary at `dist/idonthavespotify` | defined, not run here |
| `bun run build:workers` | edge bundle at `dist/workers.js` (fetch backend, console logger) | defined, not run here |
| `bun run typecheck` | `tsc --noEmit` (self-host) + workers tsconfig | verified passing |
| `bun run lint` | eslint over `src` and `tests` | verified passing |
| `bun run test` | `NODE_ENV=test bun test` (offline via `HttpMock` + local snapshots) | verified passing |
| `bun run test:ci` | regenerates snapshots from live URLs, then tests | defined, not run here (needs network; snapshots are gitignored but present in a set-up tree) |

CI: PRs run `tests.yml` (loads `.env.test` into env, then `test:ci`, plus asset/binary/edge build, native-import audit, and binary smoke); pushes to `main` run `deploy.yml` (typecheck, lint, edge build + audit, `wrangler deploy`).

## Conventions and easy-to-miss constraints

- Import alias `~/*` maps to `./src/*` (see `tsconfig.json`); keep it in new imports.
- JSX is nano-jsx classic runtime (`jsxFactory Nano.h`); client controllers stay plain `.js`.
- htmx is pinned via a CDN script tag in `src/views/layouts/main.tsx`; Stimulus is bundled into `public/assets/index.js` by `build.config.ts`.
- `public/assets/*.js|css` are build artifacts and gitignored; never edit them, rebuild instead.
- `tests/mocks/**/*.html|json` are gitignored snapshots: if tests fail with missing-mock errors on a fresh clone, run `bun run test:mocks:fetch` (live network) before `bun run test`.
- Always run tests through `bun run test`, not bare `bun test`.
- The sample track (`SAMPLE_LINK` in `src/views/pages/home.tsx`) is always a Spotify URL, and the README screenshot (`docs/screenshot.png`) is always that sample track's result card. Change both together when the sample changes.
- Outbound HTTP must go through `HttpClient` so tests can stub it; no direct `fetch`/`impit` calls in adapters or parsers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sjdonado/idonthavespotify](https://github.com/sjdonado/idonthavespotify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
