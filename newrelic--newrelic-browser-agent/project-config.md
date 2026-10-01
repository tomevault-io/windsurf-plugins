---
trigger: always_on
description: Shared, committed instructions for working on the New Relic Browser Agent
---

# CLAUDE.md

Shared, committed instructions for working on the New Relic Browser Agent
with Claude Code. This file applies to every engineer's session — it is not
personal configuration (personal preferences live in the gitignored
`.claude/settings.local.json` and are never committed here).

For product/onboarding context, see [README.md](README.md),
[DEVELOPING.md](DEVELOPING.md), and [CONTRIBUTING.md](CONTRIBUTING.md) — this
file doesn't repeat them, it covers what makes Claude's output here
predictable and reviewable on the first pass.

## Build & test commands

| Task | Command |
| ---- | ------- |
| Install | `npm ci` |
| Build CDN bundle locally | `npm run cdn:build:local` |
| Rebuild on every change | `npm run cdn:watch` |
| Full build (CDN + npm + test builds) | `npm run build:all` |
| Serve local test assets/agent | `npm run test-server` |
| Lint | `npm run lint` (`npm run lint:fix` to auto-fix) |
| Unit tests | `npm run test:unit` |
| Component tests | `npm run test:component` |
| Single jest file | `npm run test:unit -- <path>.test.js` |
| Type tests | `npm run test:types` |
| e2e (wdio), single spec | `npm run wdio:smoke -- --no-retry tests/specs/<path>.e2e.js` |

`wdio` never builds the agent for you — rebuild before (re-)running specs.
Most specs only need `cdn:build:local`/`cdn:watch`, but anything loading
`test-builds/*-wrapper/**` (e.g. `tests/specs/npm/*.e2e.js`) needs the full
`build:all`, since those pages run against the packaged npm tarball, not the
CDN bundle. See [testing.md](.claude/rules/testing.md) for the full breakdown.

## Coding style

- ESLint (`standard` + `sonarjs`, config in [.eslintrc.js](.eslintrc.js)) is
  authoritative — run `npm run lint` rather than guessing at style.
- `src/**/*.js` forbids `console.*` (`no-console: error`) — use the agent's
  own logging/warning utilities instead.
- Add JSDoc (`@param`, `@returns`, `@type` on non-obvious fields) to exported
  functions/classes and to any non-trivial internal function or complex
  variable. This isn't just documentation here: [tsconfig.json](tsconfig.json)
  has `allowJs`+`declaration`+`emitDeclarationOnly` set, so the published
  `.d.ts` types (`npm run npm:build:types`) are generated directly from
  JSDoc on the `.js` source — missing or wrong JSDoc on a public API
  produces missing or wrong published types, not just missing comments.
- When a change touches a **primary interface** — a public API
  (`src/loaders/api/*.js`), anything exposed on the `newrelic`/`NREUM`
  globals ([src/common/window/nreum.js](src/common/window/nreum.js)), or a
  registered-entity-style interface (`src/interfaces/*.js`) — add/update a
  dedicated JSDoc typings file for the shape, following the existing
  pattern: a `*-api-types.js`/`*-types.js` file with `@typedef {Object} X` +
  `@property` entries (e.g.
  [src/loaders/api/register-api-types.js](src/loaders/api/register-api-types.js)),
  imported elsewhere via `@typedef {import('./register-api-types').X}`
  (e.g. [src/interfaces/registered-entity.js](src/interfaces/registered-entity.js)).
  Don't just inline the shape as an untyped object literal — these types
  flow straight into the published `.d.ts`, and consumers (including other
  New Relic teams building on `register()`) depend on them being accurate.
- Copyright headers on `src/**/*.js` are inserted/updated automatically by
  the pre-commit hook ([.husky/pre-commit](.husky/pre-commit)). Don't
  hand-write or hand-edit them.
- New `warn()` calls need a matching numbered entry in
  [docs/warning-codes.md](docs/warning-codes.md) — the pre-commit hook blocks
  the commit otherwise (`npm run check:warning-codes`).
- Any new supportability metric (a new string sent via
  `SUPPORTABILITY_METRIC_CHANNEL`/`handle(..., ['Category/Path/Name', ...])`)
  needs a matching entry added to
  [docs/supportability-metrics.md](docs/supportability-metrics.md), grouped
  under the relevant feature heading with a `<!--- description ---> ` comment
  above it, following the file's existing format. This is **not**
  CI-enforced today (no equivalent of `check:warning-codes` exists for it),
  so it's easy to silently skip — treat it as required anyway.
- Match existing patterns in the file/directory you're editing over
  introducing a new abstraction, especially in `src/common` and `src/features/*`.

## Build size: loader vs. aggregate

The agent ships in two parts with very different cost profiles:

- **Loader** (`src/loaders/**`, each feature's `instrument/` folder) — inlined
  directly into the page's HTML/head, downloaded and parsed on every
  pageview before the page can do anything else. Every byte here is on the
  critical path.
- **Aggregate** (each feature's `aggregate/` folder) — lazy-loaded as a
  separate chunk after the page has already loaded, off the critical path.

When a change (new logic, a new dependency, an added code path) can
reasonably live in either half, **prefer putting it in aggregate over
loader**, even if that means slightly more plumbing (e.g. deferring work
across the instrument/aggregate boundary via the event emitter) than putting
it inline in `instrument/`. Loader-side growth affects page load for every
site running the agent; aggregate-side growth doesn't. CI reports bundle size

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [newrelic/newrelic-browser-agent](https://github.com/newrelic/newrelic-browser-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
