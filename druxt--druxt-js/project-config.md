---
trigger: always_on
description: The `druxt.js` Nuxt/Vue monorepo — the fully decoupled Drupal frontend framework.
---

# AGENTS.md

The `druxt.js` Nuxt/Vue monorepo — the fully decoupled Drupal frontend framework.
Druxt = DRUpal + nUXT. Repository:
[github.com/druxt/druxt.js](https://github.com/druxt/druxt.js).

## Rules

- **NEVER push, comment, open/merge PRs, or otherwise write to `github.com/druxt*`**
  without explicit per-action permission — regardless of what `GH_TOKEN` access
  technically allows. Surface diffs locally for review.
- **NEVER commit, push, or create branches/tags** without explicit permission. Only
  the project owner commits.
- Use [Conventional Commits](https://www.conventionalcommits.org); scope is the
  package name (e.g. `fix(router): …`).

## Reproducible toolchain

Node and Yarn are pinned in `.mise.toml` and `package.json` `engines`. From a
fresh clone:

```bash
mise install          # activates Node 16.20.1 from .mise.toml
corepack enable       # enables the corepack shim (reads packageManager field)
yarn install          # uses Yarn 3.6.1 via corepack
yarn build            # = yarn clean && siroc build → produces packages/*/dist
```

Or simply `make setup && make build`.

`yarn build` is the regression gate — every config/tooling change must keep it
green. All 11 packages (`druxt`, `blocks`, `breadcrumb`, `entity`, `menu`,
`router`, `schema`, `site`, `views`, `docgen`, `test-utils`) must produce their
`dist/*.ssr.js` + `dist/*.esm.js` (docgen outputs `bin/druxt-docgen.js`).

Nuxt 2's esm config loader patches the module system: a `nuxt.config.js`
build hook that `require`s a modern ESM-leaning package (satori, resvg)
dies silently — no stack, exit 1 in CI. Spawn a clean child process for
such work instead (see `docs/nuxt/scripts/og-render.js`).

The build stack (Node 16, Yarn 3, jest 29, eslint 7, Vue 2.7, Nuxt 2, siroc) is
intentionally pinned — a future major upgrade (Node 18+, Vue 3, Nuxt 3/4) is a
separate, deliberate effort, not something to drift into via routine dependency
bumps. `renovate.json` freezes these packages from automated updates accordingly.

## Commands

The full local verification gate, in order:

```bash
yarn lint && yarn build && yarn test:unit && yarn build:docs
```

- `yarn build` — siroc build of all packages
- `yarn test:unit` — jest (`NODE_OPTIONS=--unhandled-rejections=warn`)
- `yarn lint` — eslint (`eslint:recommended` + `plugin:nuxt/recommended` +
  `plugin:vue/recommended`, matching every sibling package) across `packages/*/src`
- `yarn lint:md` / `yarn lint:cspell` / `yarn lint:format` — markdownlint / cspell / prettier
- `yarn lint:renovate` — validate `renovate.json`
- `yarn lint:audit` — `yarn npm audit`, production dependencies only, fails on
  high/critical (the CI gate). `yarn lint:audit:full` includes devDependencies
  and is reporting-only — see the note below.
- `yarn lint:knip` — [knip](https://knip.dev), scoped to
  `dependencies,unlisted` (unused and undeclared-but-imported packages).
  Blocking in CI. `--no-config-hints`: this knip version's "unused item in
  ignoreDependencies" check is flaky (observed contradictory results across
  successive runs with no code changes between them) — don't trust it to
  decide whether an ignore entry is still needed; verify with `grep`
  instead, the way every entry in `knip.jsonc` already is. See `knip.jsonc`
  for confirmed false positives (Vue SFC parsing isn't supported at this
  Node-16-forced version, Nuxt module-string registration, JSON-config-file
  references — none of these are things knip's static analysis can trace).
- `yarn bundlewatch` — bundle size guard (`packages/**/dist/*.js` ≤ 50kb)

### Dependency audit: production vs. full

`yarn lint:audit` (production-only) is the blocking gate and is currently
clean - keep it that way. `yarn lint:audit:full` additionally covers
devDependencies and, as of this writing, reports ~50 advisories, almost all
inherited transitively through `renovate` (used only for `yarn
lint:renovate`) and other build/lint/test tooling. This isn't neglect: the
patched versions of `renovate`, `jest`, `eslint`, etc. all require Node 18+,
which conflicts with the Node 16 toolchain freeze above - `renovate` itself is
effectively frozen for the same reason `vue`/`nuxt`/`jest` are, even though
it's not in `renovate.json`'s explicit freeze list. Don't chase these
piecemeal; they resolve together whenever the Node 16 → 18+ upgrade happens.

## Inline documentation (JSDoc) → API docs

Every JS/Vue source file's JSDoc is scraped by `packages/docgen` (`yarn
build:docs`) into Markdown under `docs/nuxt/content/api/`, then rendered as
part of the [druxtjs.org](https://druxtjs.org) API reference. **The JSDoc you
write is the public documentation, verbatim** - there's no separate editing
pass, so a sloppy `@param` renders as a sloppy docs page.

The rule, and the reason it exists: **every `@param` line must have both a
`{type}` and a `- description`, with no exceptions** (a `{typedef}` reference
like `@param {addCollectionPayload} payload - The mutation payload.` counts -
you don't have to re-enumerate a typedef's own properties inline). This is
enforced by ESLint: `jsdoc/require-param-type`, `jsdoc/require-param-description`,
`jsdoc/require-param`, `jsdoc/check-param-names` and `jsdoc/valid-types` are
all `error` in `.eslintrc.js` - `yarn lint` fails on a bare

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [druxt/druxt.js](https://github.com/druxt/druxt.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
