---
trigger: always_on
description: Welcome to the Etherpad project. This guide provides essential context and instructions for AI agents and developers to effectively contribute to the codebase.
---

# Agent Guide - Etherpad

Welcome to the Etherpad project. This guide provides essential context and instructions for AI agents and developers to effectively contribute to the codebase.

## Project Overview
Etherpad is a real-time collaborative editor designed to be lightweight, scalable, and highly extensible via plugins.

## Technical Stack
- **Runtime:** Node.js >= 22.12.0
- **Package Manager:** pnpm (>= 11.0.0)
- **Languages:** TypeScript (primary for new code), JavaScript (legacy), CSS, HTML
- **Backend:** Express.js 5, Socket.io 4
- **Frontend:** Legacy core (`src/static`), Modern React UI (`ui/`), Admin UI (`admin/`)
- **Database:** ueberdb2 abstraction (supports dirtyDB, MySQL, PostgreSQL, Redis)
- **Build Tools:** Vite (for `ui` and `admin`), esbuild, tsx
- **Testing:** Mocha (backend), Playwright (frontend E2E), Vitest (unit)
- **Auth:** JWT (jose library), OIDC provider

## Directory Structure
- `src/node/` - Backend logic, API handlers, database models, hooks
- `src/static/` - Core frontend logic (legacy jQuery-based editor)
- `src/static/js/pluginfw/` - Plugin framework (installer, hook system)
- `src/tests/` - Test suites (backend, frontend, container)
- `ui/` - Modern React OIDC login UI (Vite + TypeScript)
- `admin/` - Modern React admin panel (Vite + TypeScript + Radix UI)
- `bin/` - CLI utilities, build scripts, plugin management tools
- `bin/plugins/` - Plugin maintenance scripts (checkPlugin.ts, updateCorePlugins.sh)
- `doc/` - Documentation (VitePress + Markdown/AsciiDoc)
- `local_plugins/` - Directory for developing and testing plugins locally
- `var/` - Runtime data (logs, dirtyDB, etc. - ignored by git)

## Quick Start

```bash
pnpm install                                    # Install all dependencies
pnpm run build:etherpad                         # Build admin UI and static assets
pnpm --filter ep_etherpad-lite run dev          # Start dev server (port 9001)
pnpm --filter ep_etherpad-lite run prod         # Start production server
```

## Core Mandates & Conventions

### Coding Style
- **Indentation:** 2 spaces for all files (JS/TS/CSS/HTML). No tabs.
- **TypeScript:** All new code should be TypeScript. Strict mode is enabled.
- **Comments:** Provide clear comments for complex logic only.
- **Backward Compatibility:** Always ensure compatibility with older versions of the database and configuration files.

### Internationalisation (i18n) — Mandatory
- **Every user-facing string MUST go through i18n.** That means JSX text, `placeholder`, `title`, `aria-label`, `alt`, toast/alert titles, `<option>` labels, error messages, breadcrumbs, empty-state copy — anything a user can see or hear via a screen reader.
- **React (admin SPA):** use `<Trans i18nKey="…"/>` for JSX text and `t('…')` for attribute values. The `t` comes from `useTranslation()`.
- **Pad UI (legacy):** use `data-l10n-id="…"` (html10n) — never bind `window._` directly to `html10n.get` and call it (it's unbound; returns `undefined`).
- **String keys live in `src/locales/en.json`.** Other locales sync from translatewiki on its own cadence — never hand-edit non-EN locale files.
- **Reuse existing keys before inventing new ones.** Check `src/locales/en.json` and the relevant plugin's `static/locale/en.json` (e.g. `admin/public/ep_admin_pads/en.json`) for an existing match. Duplicating a key like `ep_admin_pads:ep_adminpads2_last-edited` as a fresh `admin_pads.col.last_edited` fragments the translation surface for translatewiki.
- **Naming:** dot-namespaced, kebab-or-underscore-cased: `admin_plugins.subtitle`, `admin_pads.filter.all`. Group by page/feature.
- **Pluralisation:** use i18next's `_one`/`_other` suffix forms with `t('key', {count: n})` — never `n > 1 ? 'X items' : '1 item'`.
- **Locale-aware formatters:** pass a sanitised locale to `Intl.*` / `toLocaleString`. `i18n.language` is influenced by `?lng=` (user-controlled) and a malformed tag throws `RangeError`. Use a `sanitizeLocale()` helper that normalises `_`→`-` and validates via `Intl.DateTimeFormat.supportedLocalesOf()`, falling back to `'en'`.
- **`defaultValue:` in `t()` is for safety, not a substitute** for adding the key to `en.json`. If you're tempted to inline English with `defaultValue:` and skip the key, you're shipping a future-broken translation.
- **Hard prohibition:** literal German / French / Spanish / any non-English in JSX. The denylist test at `src/tests/backend-new/specs/admin-i18n-source-lint.test.ts` enforces this for the admin SPA — extend it when adding new admin files or new known-bad words.

### Accessibility (a11y) — Mandatory
- **Icon-only buttons MUST have `aria-label` AND `title`** (both — screen readers prefer `aria-label`; hover users get `title`). Lucide icons inside a button are not text content. Both labels must be `t('…')`-localised.
- **Sort controls are focusable.** A `<select>` plus a paired direction toggle is fine; a clickable column header is fine; "click invisible part of the row to sort" is not. When restyling, never strip a direction toggle without adding back an equivalent.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ether/etherpad](https://github.com/ether/etherpad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
