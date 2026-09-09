---
trigger: always_on
description: **Source of truth for AI coding tools.** There is no separate Copilot / Cursor / Gemini / Windsurf rules file in this project — use this file and the **scoped `AGENTS.md` files under `app/` and `migrations/`** (and `tests/` when present).
---

# AGENTS.md — FlightPHP Skeleton

**Source of truth for AI coding tools.** There is no separate Copilot / Cursor / Gemini / Windsurf rules file in this project — use this file and the **scoped `AGENTS.md` files under `app/` and `migrations/`** (and `tests/` when present).

**Humans coding by hand:** start with **[README.md](README.md)**. You do not need this file to ship features; it keeps hand-written and generated code on the same pattern.

**Security:** deliberate security rules live in **[SECURITY.md](SECURITY.md)**. Follow that file for auth, secrets, headers, XSS/CSRF, and reporting. Do not invent security policy in random comments.

---

## How to use this file (routing)

1. Read **this root file** for boot flow, global principles, DI, config, and “what not to do.”
2. When editing code under a directory that has its own `AGENTS.md`, **also read that file** before changing or adding files there.
3. Prefer the **nearest** scoped file for local conventions; root rules still apply (especially no `Flight::` in app layer, no `$_ENV` outside boot).

| If you are working on… | Read |
|------------------------|------|
| Controllers | [app/Controller/AGENTS.md](app/Controller/AGENTS.md) |
| Middleware | [app/Middleware/AGENTS.md](app/Middleware/AGENTS.md) |
| Models (ActiveRecord) | [app/Model/AGENTS.md](app/Model/AGENTS.md) |
| Twig templates | [app/views/AGENTS.md](app/views/AGENTS.md) |
| Runway CLI commands | [app/commands/AGENTS.md](app/commands/AGENTS.md) |
| Bootstrap / routes / services / config.php | [app/config/AGENTS.md](app/config/AGENTS.md) |
| Utils classes (`Config`, `Env`, …) | [app/Utils/AGENTS.md](app/Utils/AGENTS.md) |
| SQL migrations | [migrations/AGENTS.md](migrations/AGENTS.md) |
| PHPUnit tests | [tests/AGENTS.md](tests/AGENTS.md) |
| Security-sensitive changes | [SECURITY.md](SECURITY.md) |

If a directory has no scoped file yet, follow root rules + Flight docs; do not invent a second layout.

---

## Boot flow

```
public/index.php
  → app/config/bootstrap.php
      → vendor/autoload.php
      → App\Utils\Env::load(.env) → $_ENV
      → $app = Flight::app()
      → $fileConfig = require config.php          # literals only (Runway-safe)
      → $merged = Config::mergeEnv($fileConfig, $_ENV)  # env wins for mapped keys
      → $config = new App\Utils\Config($merged)
      → apply flight.* + timezone + CSP nonce from Config
      → app/config/services.php
          Tracy, SimplePdo, Twig, Session,
          Dice + Engine substitutions + shared services,
          registerContainerHandler
      → app/config/routes.php
      → $app->start()
```

---

## Principles

1. **One pattern per concern** — hand-written and AI output must look the same.
2. **No invented Flight APIs** — grep `vendor/flightphp/core` and use [docs](https://docs.flightphp.com) / MCP. If unsure, check docs.
3. **No `Flight::` facade in app layer** — controllers, middleware, models, view logic. Inject `flight\Engine` and services. Bootstrap/services may use `Flight::app()`.
4. **No `$_ENV` / superglobals** outside bootstrap + `App\Utils\Env` / `Config::mergeEnv`. Controllers use `Config` and `$app->request()`.
5. **Namespaces are `App\…`** — `App\Controller`, `App\Middleware`, `App\Model`, `App\Utils`, `App\Command`.
6. **Data path** — **ActiveRecord** for models; **SimplePdo** for connection, migrations, raw SQL.
7. **Views** — **Twig only** under `app/views/`. `$app->render('name', $data)` maps to Twig.
8. **Docs teach APIs; this repo teaches layout** — adapt one-file `Flight::` demos into this tree (see README “Flight docs ↔ this skeleton”).
9. **Security is explicit** — see SECURITY.md; do not weaken headers, leak secrets, or trust client input.

---

## Namespaces & layout

| Layer | Namespace / path |
|-------|------------------|
| Controllers | `App\Controller\…` → `app/Controller/` |
| Middleware | `App\Middleware\…` → `app/Middleware/` |
| Models | `App\Model\…` → `app/Model/` |
| Utils classes | `App\Utils\…` → `app/Utils/` |
| CLI commands | `App\Command\…` → `app/commands/` (Runway scans this path) |
| Views | `app/views/*.twig` (not PHP classes) |
| Framework | `flight\…` (unchanged) |

Composer PSR-4: `"App\\": "app/"`. Directory names **must** match case on Linux.

**Commands exception:** Runway discovers `app/commands/*.php` by glob and `require`s them. Keep project commands there with namespace `App\Command`.

---

## How to add… (summary)

| Task | Where | Details |
|------|--------|---------|
| Route | `app/config/routes.php` | Prefer `[Controller::class, 'method']` so Dice builds the controller |
| Controller | `app/Controller/` | Constructor injection; see scoped AGENTS |
| Middleware | `app/Middleware/` | `before(array $params)` / optional `after`; attach on group/route |
| Model | `app/Model/` | Extend `flight\ActiveRecord`; pass SimplePdo |
| Migration | `migrations/` | SQLite: `YYYYMMDDHHMMSS_description.sql`; MySQL: `…_description.mysql.sql`; then `php runway migrate` |
| Config key | sample + ENV_MAP + Config | Literals in `config.php`; secrets in `.env` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flightphp/skeleton](https://github.com/flightphp/skeleton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
