---
trigger: always_on
description: Laravel publishing **package** (`austintoddj/canvas`): PHP API + React admin SPA. Not a full app. Hosts install via Composer; admin assets publish to `public/vendor/canvas`.
---

# AGENTS.md — Canvas

Laravel publishing **package** (`austintoddj/canvas`): PHP API + React admin SPA. Not a full app. Hosts install via Composer; admin assets publish to `public/vendor/canvas`.

**Docs authority:** `readme.md` = product blurb + minimal install flyer · **`docs/`** = living host manual (install, config, auth, Canvas UI, content, webhooks) · `.github/UPGRADE.md` = version-to-version breaking changes only · `.github/CONTRIBUTING.md` = contributor/PR workflow · `AGENTS.md` = agent operating rules. Do not triplicate — host how-tos go in `docs/`, not re-pasted into readme, UPGRADE, stubs, or this file.

## Working rules

- **Prefer latest PHP/Laravel:** target the newest language and framework features within this package’s supported matrix (`composer.json` / CI). Use modern APIs for new code; don’t write down-level patterns “for older hosts” unless a supported major actually requires it.
- **Laravel Boost skills:** https://github.com/laravel/boost/tree/main/.ai/laravel — use root `core.blade.php` (cross-version) **plus** the highest versioned dir present (`12/` today; `13/` when Boost ships it; never prefer `11/` for new work) **and** `skill/laravel-best-practices`. **This file wins** when Boost’s app-centric advice conflicts with package seams (Illuminate components, local FormRequest, no host scaffold).
- **Scope:** minimal diffs only. Do not refactor adjacent code, “clean up” comments, or expand scope beyond the ask.
- **Secrets:** never commit `.env`, webhook secrets, AI keys, or host credentials; never log them in tests/fixtures.
- **Runtime:** Pest runs in-package via Orchestra Testbench — no sibling Laravel app required. E2E and install-smoke need a host (`bin/e2e-prepare.sh`, `bin/install-smoke.sh`). Do not invent a full app scaffold for unit work.
- **Patterns:** copy neighboring controllers/pages/tests. Do not introduce new layers (repositories, global stores, foundation deps) without local precedent.
- **Tests:** while iterating, filter Pest/Vitest to the touched area; run `composer test -- --filter=LocalizationTest` when changing `resources/lang` or UI copy; full PR gate below before calling work done.
- **Git:** do not perform any Git actions unless the user explicitly grants permission.

## Layout

| Path                                          | Role                                                               |
| --------------------------------------------- | ------------------------------------------------------------------ |
| `src/`                                        | Package PHP (`Canvas\`), PSR-4                                     |
| `routes/web.php`                              | Package routes (auth + API + SPA shell)                            |
| `config/canvas.php`                           | Published config                                                   |
| `database/migrations/`, `database/factories/` | Package schema + factories                                         |
| `docs/`                                       | Host-facing documentation (versioned with the package)             |
| `resources/js/`                               | Admin SPA (React 19, TipTap, RR v7, Tailwind 4, Headless UI)       |
| `resources/js/__tests__/`                     | Vitest unit/component tests                                        |
| `resources/lang/{locale}/app.php`             | UI catalog (17 locales; `en` is source of truth)                   |
| `resources/dist/`                             | **Committed** Vite build — hosts serve this                        |
| `resources/views/`, `resources/stubs/`        | Blade layout/mail + `canvas:ui` stubs (code only; no host manuals) |
| `tests/`                                      | Pest (PHP); `tests/e2e/` Playwright                                |
| `bin/`                                        | `preflight.sh`, `install-smoke.sh`, `e2e-prepare.sh`               |

Path alias: `@/*` → `resources/js/*` (`tsconfig.json`, `vite.config.ts`).

## Tooling

- **PHP** ≥ 8.3 (prefer newest CI matrix PHP), **Laravel** 12|13 (prefer newest major APIs; Illuminate components only — not full `laravel/framework` as a require)
- **Node** 22 (CI), **npm** + `package-lock.json` (`.npmrc`: `legacy-peer-deps=true`)
- Pest + Arch, Orchestra Testbench, Larastan **level 6**, Laravel Pint
- Vitest (happy-dom/jsdom via setup), Playwright e2e, ESLint 10 flat config, Prettier

## Commands (repo root)

### Install

```bash
composer install
npm ci
```

### Build / dev (SPA)

```bash
npm run dev              # Vite; writes resources/dist/canvas.hot
npm run build            # production → resources/dist/ (commit this)
npm run preview
```

### Lint / format / types

```bash
composer pint            # fix PHP (vendor/bin/pint)
composer pint:test       # check only
composer lint            # vendor/bin/phpstan analyse (phpstan.neon.dist)
npm run lint             # eslint . (resources/js/**/*.{ts,tsx})
npm run format           # prettier --write .
npm run typecheck        # tsc --noEmit (typescript-7)
```

### Tests

```bash
composer test            # vendor/bin/pest
composer test:ci         # parallel + junit → build/junit.xml

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [austintoddj/canvas](https://github.com/austintoddj/canvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
