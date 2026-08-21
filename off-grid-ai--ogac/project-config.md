---
trigger: always_on
description: Next.js 15 app. The AI gateway runs separately (the LiteLLM proxy at `<control-plane-host>:4000` on-prem).
---

# Off Grid Console — developer guide

Next.js 15 app. The AI gateway runs separately (the LiteLLM proxy at `<control-plane-host>:4000` on-prem).

## Engineering standards (non-negotiable)

- **SOLID + clear abstraction layers.** Isolate pure policy/logic from I/O (see `src/lib/tenancy-policy.ts` — a zero-import, unit-testable rule — vs `tenancy.ts`, its session/claims adapters). Business logic in `src/lib`, thin route handlers, swappable backends behind `src/lib/adapters`. Full rules: `docs/ENGINEERING.md`.
- **Write unit AND integration tests.** Tests live in `test/`, run with `npm test` (`node --test`, type-stripped). Unit-test the pure logic; integration-test the real wiring.
- **Use mocks very sparingly** — prefer exercising real functionality (real functions, real DB/services where feasible) so tests don't hide underlying behavior. If you're mocking a lot, the code probably needs a cleaner seam instead.
- **DRY — no copy-paste logic.** One rule lives in one place (a pure helper in `src/lib`), reused; if two surfaces need the same decision, extract it, don't duplicate it. Duplicated logic that drifts is a defect — we review for it.
- **Coverage bar (≥85%, enforced): branches, statements, lines, functions, and conditions must each be ≥85%** on the unit-testable logic layer (`src/lib` pure logic + adapters' pure paths). Measured with `npm run coverage` (c8); **enforced by the pre-push hook** (`npm run coverage:check`, `check-coverage: true`) — a push below the bar is blocked. Conditions are covered via branch coverage (each `&&`/`||`/`?:` arm exercised both ways — c8's branch metric; true MC/DC isn't a standard JS metric). Pure-I/O glue that genuinely needs live services (DB/network clients, worker entrypoints, React `.tsx`) is excluded from the denominator and verified by integration tests + build + vision instead — coverage thresholds apply to logic you can unit-test, not to glue. **Every change adds real tests; the bar only goes up.** SOLID's pure-logic-isolated-from-IO split is what makes this reachable — a file that's hard to cover usually needs a cleaner seam.
- **Navigation must live in the URL / history stack.** Every screen change or in-page navigation (opening a folder, a tab, a detail view, a modal that's a "place") MUST push a corresponding history entry — drive it from the route/`searchParams` (`useRouter`/`useSearchParams`), not local `useState`. This keeps the browser Back button coherent (Back steps out of a folder/tab, doesn't dump you off the page) and makes views deep-linkable/shareable. Client-only state for a navigational position is a bug.
- **Every module is a full CRUD management surface — not a read-only dashboard.** The console is how operators **run and maintain their systems**, so each module must let them **create, read, update, AND delete** the entities it covers, and **trigger the actions** that manage the underlying system (run an eval, run/schedule a backup, re-run/cancel an agent run, push/reload a policy, create/delete a collection, write a secret, edit a masking rule). A page that only lists/aggregates data is **the bare minimum and NOT a finished feature.** For each entity: create/edit forms with validation, delete with confirmation, proper error handling, and the write routes (POST/PATCH/DELETE) behind them — console-owned entities in the DB, external-service entities pushed through the service's API. Keep the SOLID split (pure rules in `src/lib`, thin routes, tests), but the deliverable is a working management console, end-to-end usable.

## Systems of record — READ THESE (don't keep infra knowledge in your head)

> **On-prem / fleet deployment orchestration now lives in the PRIVATE repo
> [`off-grid-ai/onprem-fleet-orchestration`](https://github.com/off-grid-ai/onprem-fleet-orchestration)**, not
> in this (public-bound) repo. The specific-deployment systems of record — `SERVER_STATE.md`,
> `SERVICE_MAP.md`, `DEPLOY.md`, the Cloudflare tunnel config, `dns-records.sh`, seed SQL, and the
> deploy scripts (`push.sh`/`fleet.sh`/`prod.sh`/`verify-integration.sh`) — were moved there. Clone
> that repo alongside this one to deploy or change infra.

Every out-of-code change to the on-prem deployment MUST be captured in one of those records (in the
private repo), in the same commit that makes the change — otherwise it's lost when the session ends.
Generic self-host tooling that stays in THIS repo: `deploy/docker-compose*.yml`, `deploy/Makefile`,
`deploy/keycloak/`, `deploy/openbao/`, `deploy/sidecars/`, `deploy/README.md`.

**`docs/ROADMAP.md`** — phases + milestones. **`docs/ENGINEERING.md`** — SOLID / ports-and-adapters rules.

## Dev

```bash
npm run dev          # start dev server
npm run build        # production build
npm run typecheck    # tsc --noEmit
npm run db:push      # push schema changes
npm run smoke        # hit each service health endpoint
```

Infra (Postgres, OpenBao, Redis, etc.) lives in `deploy/`. Start what you need:

```bash
cd deploy && make up           # full stack
cd deploy && make data         # Postgres + SeaweedFS only
cd deploy && make secrets      # OpenBao only
```

## Deploying to the on-prem fleet

**Full runbook: [`deploy/DEPLOY.md`](deploy/DEPLOY.md) — read it before deploying.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [off-grid-ai/OGAC](https://github.com/off-grid-ai/OGAC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
