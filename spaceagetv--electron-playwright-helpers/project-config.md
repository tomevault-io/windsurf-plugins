---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`electron-playwright-helpers` — an npm library of helper functions for testing Electron apps with Playwright. Published from `src/` → `dist/`. Node 18+ required (uses `structuredClone()`).

## Commands

Two separate npm projects: the library at the root, and `example-project/` (an electron-forge app used as the e2e fixture). Both need installing:

```bash
npm ci && cd example-project && npm ci
```

| Task | Command |
| --- | --- |
| Build (compile + regenerate README) | `npm run make` |
| Compile only | `npm run make:compile` |
| Unit tests (mocha + `node:assert/strict`, no Electron) | `npm run test:unit` |
| Single unit test | `npx mocha --require ts-node/register --timeout 5000 './test/**/*.ts' --exit --grep "retry"` |
| E2E tests (packages the app first via `pree2e`) | `npm run test:e2e` |
| Single e2e test | `cd example-project && npx playwright test -g "test name"` |
| E2E without repackaging | `cd example-project && npx playwright test` |
| Lint / fix | `npm run lint` / `npm run lint:fix` |
| Types | `npm run type-check` |

`npm test` runs unit then e2e. E2E needs a packaged build in `example-project/out/` — `npm run package` (electron-forge) creates it, and `findLatestBuild()` locates it at test time. On Linux/CI, e2e runs under `xvfb-run` (see `.github/workflows/e2e-ci.yml`).

**README.md is generated** — `npm run make:doc` renders `readme-template.hbs.md` + JSDoc from `src/*.ts` via jsdoc2md. Never hand-edit README.md; edit the template or the JSDoc comments.

Releases are automatic: semantic-release on `main` (and prerelease on `beta`) driven by Conventional Commit messages. Don't bump versions manually or edit `docs/CHANGELOG.md`.

## Architecture

Every helper is a thin wrapper around `electronApp.evaluate()` or `page.evaluate()`, and nearly all of them are wrapped in `retry()`. `src/index.ts` just re-exports each module.

### `retry()` is the backbone — `src/utilities.ts`

Since Electron 27, Playwright's `evaluate()` throws spurious context errors. `retry()` retries *only* errors whose message matches `retryDefaults.errorMatch` in `src/utilities.ts` — a list of substrings taken from Playwright's own messages — and rethrows everything else immediately.

**Those substrings are coupled to Playwright's internals**, specifically `rewriteError()` in `crExecutionContext.ts`, which rewrites raw CDP errors into user-facing ones. When Playwright rewords a message the list stops matching, and the symptom downstream is "sudden flakiness" after a Playwright upgrade. Read that function in the installed `playwright-core` before concluding a helper is at fault. The fix belongs in the `errorMatch` list, not in individual helpers, and should be additive — older Playwright versions may still emit the previous wording. `setRetryOptions()` / `getRetryOptions()` / `resetRetryOptions()` mutate a module-level singleton (`currentRetryOptions`), so changes are global and persist across tests until reset.

#### Two different failures hide behind those messages — only one is retryable

**Teardown (`teardownErrorMatch`) — transient, retry it.** The execution context a call was dispatched into went away. Playwright cannot recover from this on the Electron *main* process: `ElectronApplication` resolves `_nodeElectronHandlePromise` exactly once, on the first `Runtime.executionContextCreated`, to a `JSHandle` for `require('electron')`, and never re-acquires it — `_nodeSession` listens only for `Runtime.executionContextCreated` and `Runtime.consoleAPICalled`, never `executionContextDestroyed`. Pages get proper re-acquire machinery (`Frame.contextCreated/contextDestroyed` swaps in a fresh `ManualPromise` per world); `ElectronApplication` gets none. Retrying from the outside is the only lever available. Note also that `rewriteError()`'s last branch is a **catch-all**: any CDP error that is neither a JS-error-in-evaluate nor session-closed becomes "Execution context was destroyed, most likely because of a navigation", so that one string covers an unknown set of underlying protocol errors.

**Garbage-collected promise (`gcErrorMatch`) — deterministic, do NOT retry it.** Playwright sends `Runtime.callFunctionOn` with `awaitPromise: true`; V8's inspector tracks the awaited promise through a *weak* handle (`ProtocolPromiseHandler`, `v8/src/inspector/injected-script.cc`), so a promise that nothing in the target heap references gets collected before it settles and CDP answers "Promise was collected". Measured behavior on Playwright 1.62.1 / Electron 35, main process and renderer alike:

- Reproduces 100% (`evaluate(() => new Promise(() => {}))` plus allocation churn); realistic async evaluates are immune (0 failures in 300 iterations with forced GC at the await point) because Electron's native promises are held by a strong `v8::Global`.
- **The callback body already ran** — synchronous work and timer-rooted async side effects all completed; only the reply was lost. Retrying re-fires those side effects.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spaceagetv/electron-playwright-helpers](https://github.com/spaceagetv/electron-playwright-helpers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
