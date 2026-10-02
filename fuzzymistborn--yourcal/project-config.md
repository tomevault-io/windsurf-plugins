---
trigger: always_on
description: Monorepo: `shared` (types) / `server` (Fastify API) / `client` (Vue 3 SPA),
---

# AGENTS.md

Monorepo: `shared` (types) / `server` (Fastify API) / `client` (Vue 3 SPA),
npm workspaces. Package/workspace names are `yourcal` / `@yourcal/*`. The
CalDAV server is the sole identity provider — no local user store.

Verification discipline throughout: features are driven against a real
CalDAV server with curl, not just typechecked. Automated tests
(`server/src/**/*.test.ts`, `npm test` / vitest, run in CI) cover the
mapper, recurrence, edit scopes, import/export, bounds, SSRF, privileges,
the SQLite store, and route handlers — but they don't exercise the CalDAV
server's own RRULE/ETag validation, which is still only checked by hand.

## Open items

- **Conflict-handling UX** — not built.
- **`GET /api/calendars/:id/sync`** — `DavCalendarStore.syncCalendar()` is
  implemented but no route exposes it and it's never been tested against a
  real server.
- **VTODO/task list UI** — deliberately out of scope. `Calendar.supportsTasks`
  is discovered and plumbed through the store layer but has no UI and should
  stay that way. Attendees/attachments are also out of scope.
- **Baikal coverage** — real Baikal 0.11.1 has only been exercised for
  sharing and plain event CRUD. RRULE/EXDATE/timezone edge cases are
  Radicale-only-verified. Baikal share **management** (list/update/revoke,
  as opposed to creating a share) is implemented but never spike-tested (no
  PHP on this host).
- **Frontend** has only been clicked through piecemeal in a real browser;
  no systematic pass over every dialog / drag / resize path.
- **Docker** image is unbuilt/unrun (no Docker on this host) — see below.

Done, each with a section below: search, ICS import, WebCal/ICS
subscriptions, recurring-event UI (interval / `BYDAY` picker / end
condition, ordinal `BYDAY`, `RDATE`, override preservation), timezone
support, VALARM reminders, per-event `COLOR`, calendar create / rename /
delete, read-only calendars, ICS export, calendar sharing (create / accept
/ unsubscribe / owner-side management), SQLite read-cache, calendar sort,
undo toast, agenda / year / mini-month navigator, duplicate event /
copy-paste, print view, 12h/24h time format setting, view options /
new-event defaults, dark mode, auto-refresh.

## Toolchain pins

**`typescript` is pinned to `^7.0.2`** (dependabot PR #9). Two parts of the
toolchain don't support TS7 yet:

- **`@typescript-eslint`** peer-caps at `typescript <6.1.0`
  ([issue #10940](https://github.com/typescript-eslint/typescript-eslint/issues/10940),
  "1-2 major releases" out). Workaround: CI runs `npm ci --legacy-peer-deps`.
  Treat lint output with suspicion on TS7-only syntax. There's also no
  `eslint.config.js` in the repo at all, so `npm run lint` currently fails
  regardless — separate pre-existing gap.
- **`vue-tsc`** hard-crashes on TS7 (`ERR_PACKAGE_PATH_NOT_EXPORTED` for
  `./lib/tsc`, removed in TS7 —
  [vuejs/language-tools#6124](https://github.com/vuejs/language-tools/issues/6124),
  [#5381](https://github.com/vuejs/language-tools/issues/5381)). Workaround:
  `client`'s `build` script is just `vite build`; type-checking moved to a
  separate `npm run typecheck -w client` that **nothing runs automatically**
  — client type errors are caught by neither `build` nor CI until this is
  revisited.

**Revisit both** when the upstreams ship TS7 support: drop
`--legacy-peer-deps` and put `vue-tsc -b` back in front of `vite build`.

**`@fullcalendar/*` pinned to `^6.1.21`** across all five packages.
Dependabot PR #9 bumped only `core`/`vue3` to `7.0.2`; `daygrid`/
`interaction`/`timegrid` have no stable v7 (only `-rc`), and FullCalendar
requires one matched major across the family — the mixed set broke module
resolution at build time. Reverting also required regenerating
`package-lock.json` from scratch (a plain reinstall kept a stale nested
`core@7.0.2`). Revisit when all the packages have a matching stable v7.
`.github/dependabot.yml` now `ignore`s `@fullcalendar/*`
`version-update:semver-major`, so the grouped PR stops re-proposing a
half-family v7 bump every week; drop that ignore when bumping the family
together. (The family is now seven packages — `list` and `multimonth`
were added for the agenda/year views.)

## Dev CalDAV servers (no Docker on this host)

Both `.dev-radicale/` and `.dev-baikal/` are git-untracked scratch state.

### Radicale (3.7.7)

```
python3 -m venv .dev-radicale/venv
.dev-radicale/venv/bin/pip install radicale bcrypt
.dev-radicale/venv/bin/python -m radicale --config .dev-radicale/config/config
```

Binds `127.0.0.1:5232`, serves DAV at the root. htpasswd auth
(`.dev-radicale/config/users`), filesystem storage under
`.dev-radicale/data/`. A calendar must be created per user with a raw
`MKCALENDAR` (Radicale doesn't auto-create one).

Config additions for sharing (all git-untracked): a `[rights]` section
(custom `from_file` rights reimplementing `owner_only` for every user plus
one legacy spike rule) and a `[sharing]` section (`type = files`,
`collection_by_map = true`, `permit_create_map = true`, db under
`.dev-radicale/data/collection-db/`). Users: `testuser`/`testuser`,
`shareduser`/`shareduser`.

Reusable fixtures — two fully-accepted map shares from `testuser` to
`shareduser`: Personal at `/shareduser/testuser-personal/`, Work at

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FuzzyMistborn/YourCAL](https://github.com/FuzzyMistborn/YourCAL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
