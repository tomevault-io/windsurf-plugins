---
trigger: always_on
description: Notes for anyone — human or agent — working in this repository. `README.md` is the
---

# CLAUDE.md

Notes for anyone — human or agent — working in this repository. `README.md` is the
user-facing documentation; this file is the parts that are easy to break.

## What this is

A personal dashboard for a trusted LAN. One idea carries the whole design: **a
widget is a TypeScript function on disk**. The platform decides *when* to call it
(clock, MQTT message, webhook), validates *what* it returns against a typed
schema, and pushes the result to every open tab over SSE. There is no plugin
format and no config-driven renderer path, so summing a field, joining two APIs
and remembering what you already saw are all the same amount of work.

Widget code is **trusted**. Isolation exists to contain bugs, not attackers. There
is one optional login, over `/admin` only (`ADMIN_PASSWORD` in `.env`) — it keeps
the panel out of reach of whoever walks past the wall display, and it is not a
hardening story. Multi-user is still out of scope; see the bottom.

## Commands

```bash
npm run dev        # node --watch on :8080 + vite on :5273 (proxies /api)
npm test           # vitest; MQTT suites skip themselves without a broker
npm run typecheck  # both projects; run this, tsc catches what vitest can't
npm run build      # web bundle only — the server has no build step

docker compose -p n-dashboard-dev -f docker/compose.dev.yml up -d   # broker for tests
```

## Repo map

```
.claude/skills/ write-widget — the widget-authoring workflow, for agents
src/shared/    ctx.d.ts (the widget API), kind schemas, duration parsing
src/server/    express, worker pool, scheduler, connections, ndjson store
src/web/       react — dashboard (kept light) + admin (Monaco, lazy-loaded)
docker/        Dockerfile, both compose files, broker configs
docs/          the long-form guides linked from README.md, plus the Pages page
examples/      seeded into data/ disabled on first run
data/          runtime state, gitignored
```

## Invariants

Break one of these and it usually fails somewhere far away from the change.

**`src/shared/ctx.d.ts` is the single source of truth for the widget API.** It is
served verbatim at `/api/widget-api.d.ts` for Monaco, imported as types by both
the server and the web app, and quoted in the README. It must stay a plain
ambient `declare module` — no top-level import or export — or the ambient
declaration stops applying.

**Kind schemas are declared `z.ZodType<KindDataMap[K]>`.** That is what makes a
disagreement between `ctx.d.ts` and a runtime validator a compile error rather
than a widget that validates against a shape the editor never offered. Keep it.

**`KIND_CATALOG` and the web `RENDERERS` map are declared over the full
`KindName` union.** Adding a type to `ctx.d.ts` without registering a schema and
a renderer is therefore a compile error. This is on purpose.

**Results are validated on the main thread, after the worker returns.** Never
inside the worker: a widget must not be able to vote on whether its own output is
valid.

**Runtime aliases are package.json `imports` (`#shared/*`, `#server/*`), not
tsconfig `paths`.** tsconfig paths are a type-only fiction that Node cannot
resolve; subpath imports resolve natively in every thread, which is what lets
worker threads run with no loader registered. Consequences:

- Server and shared code must use **explicit `.ts` extensions** in relative and
  `#alias` imports.
- `node --watch-path=./src` is deliberate. Without the path restriction, `--watch`
  also watches the dynamically imported `data/widgets/*/widget.ts` and restarts
  the server every time the admin panel saves a widget.

**Everything obeys `erasableSyntaxOnly`** — no `enum`, no `namespace`, no
constructor parameter properties, anywhere. Widget code is run by Node's type
stripping, and the rest of the project matches so the rules are uniform.

**Widget modules are cache-busted with `?v=<mtimeMs>`.** A worker that has
imported `widget.ts` caches it for the life of the thread, so without the query an
edited widget would keep running its old code until restart.

**Secrets and `ctx.state` ride along in the invoke message.** They are small and
known up front, and shipping them is what lets `ctx.secret()` and
`ctx.state.get()` stay *synchronous* for widget authors despite the thread
boundary. Only `mqtt`, `google` and `history` calls round-trip to the main thread.

**The admin guard is fail-closed: `PUBLIC` in `src/server/auth.ts` is the entire
list of ungated API routes.** A route added anywhere under `/api` is protected the
moment `ADMIN_PASSWORD` is set, until somebody names it there — the alternative,
listing what to protect, ships an open route every time a file is added. The list
is read-only: a signed-out board draws itself and a webhook fires, and every
write — layout, editing, theme, refresh — needs a session. That is what the
dashboard header's `signedIn` checks mirror; they hide buttons that would 401, they
do not decide anything. `authRouter` is mounted *before* the guard in `index.ts`,
which is why the login endpoints work while signed out; they are not in `PUBLIC`.
The React `AdminGate` is likewise a convenience so the panel shows a form instead
of a screen of failed requests — it is not the boundary, and nothing may start
depending on it as one.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devapro/n-dashboard](https://github.com/devapro/n-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
