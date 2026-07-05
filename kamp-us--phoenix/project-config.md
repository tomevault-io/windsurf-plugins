---
trigger: always_on
description: A multi-app, multi-worker repo: one Cloudflare Worker per app under `apps/`
---

# phoenix

kamp.us, reborn.

A multi-app, multi-worker repo: one Cloudflare Worker per app under `apps/`
(`web` is the only app today), each its own package + stack + stage (ADR
0057). React 19 + Effect + fate. HTTP via Effect `HttpRouter` / `HttpApiBuilder`
(ADR 0027). Durable Objects authored on alchemy's Effect DO model (ADR 0028).

## Architecture

- **One worker per app.** Each `apps/<app>` is its own pnpm package owning its own
  `alchemy.run.ts` stack + per-app stage, reusing the account-global state store and
  the four CI secrets — no second bootstrap (ADR 0057). `apps/web` is the only worker
  today; the structure fans out as apps are added.
- **`apps/web`** serves both the SPA (via `assets` binding) and the API.
- The data layer is [fate](https://github.com/usirin/fate)'s native protocol: `/fate` serves data views, `/fate/live` drives live views over SSE. Other backend routes live under `/api/*`.
- Frontend is React 19 + Vite, built into `dist/client`.
- DOs are bindings on the same worker: a single unified `LiveDO` (ADR 0037, on the Effect DO model of ADR 0028) plays both the connection and topic roles to power the fate-live SSE fan-out (ADRs 0023/0025). Add more DOs per feature.

```
phoenix/
├── apps/                    # one worker per app, each its own package + stack (ADR 0057)
│   └── web/                 # @kampus/web — the only worker today
│       ├── worker/          # worker entry + backend code
│       ├── src/             # React frontend
│       └── alchemy.run.ts   # this app's alchemy stack (replaces wrangler.jsonc)
├── packages/                # shared internal packages
├── infra/                   # standalone stacks: ci-credentials (one-shot CI-token provisioner), depo (internal asset store/CDN — designed, ADR 0144)
└── pnpm-workspace.yaml
```

## Commands

```bash
pnpm install
cp apps/web/.env.example apps/web/.env   # first run only — local dev env (gitignored)
pnpm dev          # turbo-driven; two processes: `vite` (SPA/HMR) + `alchemy dev` (worker)
pnpm dev:web      # just the Vite SPA dev server
pnpm dev:worker   # just `alchemy dev` (the worker on a local workerd, offline)
pnpm build
pnpm deploy       # pnpm build && alchemy deploy (use --stage <name> for isolation)
pnpm typecheck
pnpm lint         # biome check
pnpm format       # biome check --write
```

`alchemy dev` auto-loads `apps/web/.env` (it layers a `.env` over `process.env`), so `BETTER_AUTH_SECRET` (a required `Config.redacted`, no default) and `ENVIRONMENT` come from there — copy `.env.example` → `.env` once. Production secrets are Cloudflare `secret_text` bindings set by `alchemy deploy`, never read from `.env`.

Deploy is alchemy-managed (ADR 0026–0031): `alchemy.run.ts` is the stack, there is
no `wrangler.jsonc`. `alchemy deploy --stage <name>` yields an isolated worker + D1
+ DOs per stage; CI uses the Cloudflare-hosted state store, local dev uses
`Alchemy.localState()` (offline).

## pnpm over npm

- All commands use `pnpm`.
- Never use `npx ...`; use `pnpm dlx ...`.

## Lineage

- Tech is rebuilt from `~/code/github.com/kamp-us/kampus/` (worker + DO patterns).
- Products are reborn from `~/code/github.com/kamp-us/monorepo/` (sozluk, pano, kampus).
- The shape is `kampus`, the products are `monorepo`, collapsed into the `apps/web` worker.

## Conventions

- Biome formatting: tabs, 100 col, no bracket spacing.
- **Node over Python for scripts/hooks.** Mechanical tooling lives as an Effect CLI package under `packages/` (the `epic-ledger` / `crabbox-manifest` / `leak-guard` idiom — `effect/unstable/cli`, run with `node src/bin.ts`), not an ad-hoc script or a Python hook. A pure, unit-tested core + a thin Effect bin; never a one-off `.py`.
- Effect for backend control flow; feature services are isolate-level layers, with the per-request services (`CurrentUser`, `LivePublisher`, `CurrentActor`) provided onto each handler from the validated session (ADRs 0029/0041; `CurrentActor` per ADR 0107 §7). `Auth` is a BetterAuth type alias, not the per-request session carrier — `CurrentUser` (ADR 0042) is.
- Make invalid states unrepresentable. Domain logic in domain objects.
- **Comments earn their place or die.** Code must not be buried between comments a reader pattern-matches as boilerplate and skips — a skipped comment is pure noise that rots unread. Not anti-comment: a load-bearing note is the point. But the *why* belongs in `.decisions/`, how-the-code-is-shaped in `.patterns/`; an inline comment is the surface of last resort, for a note with no other home that belongs at this exact line (a local invariant at its enforcement site, a workaround + its forcing constraint, a deliberate-looking-wrong guard, a pragma rationale). Cut separators, name-restaters, and narration of obvious control flow; collapse a docblock that re-derives an ADR's *why* to a pointer (`// See ADR NNNN`). A top-of-file docblock is fine when it states what the module is + the one non-obvious thing — not an essay re-deriving the code. Enforce with [`/deslop-comments`](.claude/skills/deslop-comments/SKILL.md).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kamp-us/phoenix](https://github.com/kamp-us/phoenix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
