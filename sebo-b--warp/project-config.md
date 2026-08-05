---
trigger: always_on
description: Guiding principles and project map for AI coding agents working on WARP. It
---

# AGENTS.md

Guiding principles and project map for AI coding agents working on WARP. It
complements (does not replace) the tracked docs in §2.

---

## 1. What WARP is

**WARP** — Workspace Autonomous Reservation Program. A hybrid-office desk (and
parking) reservation system: users book/cancel seats on floor maps, admins
manage maps, zones, groups and users. Flask backend (peewee + PostgreSQL),
hand-rolled SPA frontend (native ES modules + webpack), Materialize 2.x CSS,
native `<dialog>` modals. Installable as a PWA. Auth backends: built-in, LDAP,
Azure AD (MSAL), OIDC, SAML.

### Repo layout

```
warp/                 # Flask app package (the backend, Python)
  config.py           # all settings + WARP_ env-var parsing (single source of truth)
  db.py               # peewee models / ORM layer
  auth*.py            # auth backends (built-in, ldap, aad, oidc, saml, mellon)
  view.py             # server-rendered routes + SPA mount
  xhr/                # XHR API endpoints (bookings, plans, zones, users, groups, ...)
  sql/                # schema.sql + numbered migration_*.sql + sample_data.sql
  static/             # served assets
    theme.css         # COLOURS ONLY — see Themes/CSS
    i18n/             # en/de/fr/es/pl translation JSON
    dist/             # webpack output (GITIGNORED — never commit)
    sw.js             # PWA service worker
  templates/          # Jinja templates; headers/ is webpack HTML output (GITIGNORED)
js/                   # frontend source (JS, webpack → warp/static/dist/)
  base/style.css      # structural CSS (spacing, radius, M3 mappings)
  app/                # router.js, dialog.js (WarpDialog), modals/, i18n, theme, ...
  views/              # one module per admin/booking view
tests/                # pytest suite (pure-Python, no fixtures framework)
e2e/                  # Playwright browser suite (see §4)
containers/           # Dockerfile (prod), Dockerfile_debug (e2e), compose/, quadlet/
res/                  # demo gif, icons, check_i18n.py, perf/gen scripts
```

---

## 2. Tracked documentation

Version-controlled and authoritative — update them when behaviour changes
(see §5) and treat them as part of the codebase.

| Doc | Scope |
|-----|-------|
| [README.md](README.md) | overview, quick start, dev setup, container images |
| [FEATURES.md](FEATURES.md) | everything a user/tester/admin can do; basis for the e2e suite |
| [CONFIGURATION.md](CONFIGURATION.md) | every `warp/config.py` setting and `WARP_` env var |
| [GLOSSARY.md](GLOSSARY.md) | plain-language definitions (zone, plan, seat, assignment) |
| [PERMISSIONS.md](PERMISSIONS.md) | the authoritative access model |
| [AUTOBOOK.md](AUTOBOOK.md) | auto-book seat-picking heuristics — only relevant when changing the auto-book logic (`warp/xhr/plan.py`); no need to read it otherwise |
| [e2e/README.md](e2e/README.md) | e2e harness, how to run, test accounts, writing conventions |
| [containers/README.md](containers/README.md) | prod + debug images, compose, Podman Quadlet |

`PLAN_*.md` and `graphify-out/` are gitignored local working documents;
`CODE_REVIEW.md` is a working review log.

---

## 3. pytest (`tests/`)

Pure-Python unit tests for non-UI logic: SAML/OIDC metadata & routes, PWA,
group/zone mapping, timezone/time handling, calendar utils. No `conftest.py` /
fixtures framework — each `test_*.py` is standalone.

```sh
source .venv/bin/activate      # python venv (see README dev setup)
pip install -r requirements.txt
pytest                         # from repo root
pytest tests/test_pwa.py       # single file
```

The frontend build (`cd js && npm ci && npm run build`) is only needed to run
the app, not pytest.

External auth providers (LDAP / Azure AD / OIDC / SAML) can't be exercised by
the self-contained e2e container, so their behaviour relies on these unit tests
(`test_saml_*`, `test_oidc_*`, …); keep that coverage in step when touching
auth backends.

---

## 4. e2e tests (`e2e/`)

Browser-driven Playwright suite against the real UI, run in a self-contained
container built from `containers/Dockerfile_debug` (PostgreSQL + Flask debug)
which the harness builds and starts automatically. **Podman is available**
(preferred over docker when both exist; auto-detected).

```sh
cd e2e
npm ci
npx playwright install chromium
npm test                       # builds + starts the container automatically
npm run test:headed            # watch the browser
npm run test:ui                # Playwright UI mode
npx playwright test tests/booking.spec.ts   # single file
npm run test:officemap         # OfficeMap component suite — backend-free, own config, no container
npm run report                 # open last HTML report
```

See [`e2e/README.md`](e2e/README.md) for the full harness description. Key rules
that catch agents out:

- **e2e is e2e: drive the real UI** (click, type, navigate). Do not call the
  XHR/HTTP API directly. The only allowed backchannel is the **database** for
  test setup/reset and assertions on persisted state — and even that only when a
  real UI flow can't set up the precondition (e.g. seeding a specific row).
- Import `test`/`expect` from `../fixtures`, never from `@playwright/test`
  (the fixture resets the DB + virtual clock before each test).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sebo-b/warp](https://github.com/sebo-b/warp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
