---
trigger: always_on
description: Legacy vs NFS app migration and e2e testing (all workspaces)
---


# Legacy App & NFS App Migration and E2E Testing

Use this when working on migration between a legacy frontend and an NFS (new frontend system) app in **any workspace**, or when adding or changing e2e tests.

## App layout (when a workspace has both)

- **Legacy app**: typically `packages/app-legacy` — original Backstage frontend. Started with e.g. `backstage-cli repo start packages/app-legacy packages/backend`.
- **NFS app**: typically `packages/app` — app using the new frontend system. Default `yarn start` (repo start).

Root `package.json` scripts to support:

- `start:legacy` — run legacy app + backend.
- `start` — run NFS app (repo start).
- `test:e2e:legacy` — e2e against legacy (`APP_MODE=legacy playwright test`).
- `test:e2e:nfs` — e2e against NFS (`APP_MODE=nfs playwright test`).
- `test:e2e:all` — run both (`yarn test:e2e:legacy && yarn test:e2e:nfs`).

When changing app behavior or navigation, consider impact on **both** apps and run both e2e suites where both exist.

## APP_MODE and Playwright

- `APP_MODE=legacy` → use legacy start command (e.g. `yarn start:legacy`).
- `APP_MODE=nfs` → use NFS start command (e.g. `yarn start`).
- In `playwright.config.ts`: `const appMode = process.env.APP_MODE || 'legacy'` and `const startCommand = appMode === 'legacy' ? 'yarn start:legacy' : 'yarn start'`.

Keep artifacts per mode so results don’t collide:

- Reporter output: `e2e-test-report-${appMode}`.
- Test output dir: `node_modules/.cache/e2e-test-results-${appMode}`.

## E2E layout (workspace-agnostic)

Two valid patterns in this repo:

1. **Workspace-root e2e** — `e2e-tests/` at the workspace root. Shared utils in `e2e-tests/utils/`. Best when the same tests run against both legacy and NFS via `APP_MODE`.
2. **App-scoped e2e** — `packages/app/e2e-tests/` (and optionally `packages/app/e2e-tests/utils/`). Used by many workspaces that only have one app.

When migrating a workspace to support both legacy and NFS, consider moving e2e from `packages/app/e2e-tests/` to workspace-root `e2e-tests/` so one suite is driven by `APP_MODE`. Set `testDir: 'e2e-tests'` in Playwright config for root layout, or `testDir: 'packages/app/e2e-tests'` for app-scoped.

**Locale/config overrides**: Use a `test_yamls/` dir (under the chosen e2e dir or a known path) with `app-config-e2e-{locale}.yaml` (or similar) to override `app.baseUrl` and `backend` (port, CORS) per locale. Playwright `projects` and `webServer` entries should match those configs and ports.

## Translations and locales in E2E

- Prefer **translation keys** from the plugin (e.g. `translations.some.key`) over hardcoded UI strings so tests work in all supported locales.
- Load messages from the plugin’s translation modules (e.g. ref + locale files). Use a small util (e.g. `getTranslations(locale)`) in `e2e-tests/utils/` or `packages/app/e2e-tests/utils/`. Use eslint-disable for relative monorepo imports only where necessary.
- For template strings with placeholders, use a `replaceTemplate(template, { key: value })` helper (e.g. replacing `{{key}}`).
- For non‑default locale, switch via UI (e.g. Settings → language) in a `switchToLocale(page, locale)` helper and keep selectors/key usage locale-agnostic.

## Port mapping (multi-locale e2e)

When running multiple frontends/backends (e.g. one per locale), keep a consistent mapping so test code and configs stay in sync. Example pattern: frontend 3000, 3001, … and backend 7007, 7008, … (e.g. `backendPort = frontendPort + 4007`). Document the mapping in the workspace or in a comment in Playwright config / events helper.

## E2E helpers and quality

- **Shared logic**: Put reusable flows (navigation, panels, tables, date pickers, etc.) in `utils/` under the chosen e2e dir. Use the same helpers for legacy and NFS runs.
- **Accessibility**: Run axe (e.g. `@axe-core/playwright`) with wcag tags; attach results to `TestInfo`; filter known false positives so only real violations fail the run.
- **Timing**: After actions that trigger async or analytics, wait appropriately (e.g. a short delay or wait for network/UI) before assertions.

## Migration runbook (adding legacy + NFS to a workspace)

When a workspace currently has only one app and you need to support both legacy and NFS with shared e2e:

1. **Create or rename app packages**
   - If the current app is the new frontend: rename/copy it to `packages/app-legacy` (or create a legacy app) and keep the NFS app as `packages/app`.
   - Ensure root `package.json` has `start:legacy` (e.g. `backstage-cli repo start packages/app-legacy packages/backend`) and `start` (repo start for NFS).

2. **Add e2e scripts and APP_MODE**
   - In workspace root `package.json`: `test:e2e:legacy` = `APP_MODE=legacy playwright test`, `test:e2e:nfs` = `APP_MODE=nfs playwright test`, `test:e2e:all` = run both in sequence.

3. **Update `playwright.config.ts`**
   - `const appMode = process.env.APP_MODE || 'legacy'`.
   - `const startCommand = appMode === 'legacy' ? 'yarn start:legacy' : 'yarn start'`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [redhat-developer/rhdh-plugins](https://github.com/redhat-developer/rhdh-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
