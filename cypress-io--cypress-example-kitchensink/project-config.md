---
trigger: always_on
description: An example application demonstrating most of the [Cypress API](https://on.cypress.io/api), published to [example.cypress.io](https://example.cypress.io/). It serves two audiences at once, and changes have to work for both:
---

# cypress-example-kitchensink — AGENTS.md

## Overview

An example application demonstrating most of the [Cypress API](https://on.cypress.io/api), published to [example.cypress.io](https://example.cypress.io/). It serves two audiences at once, and changes have to work for both:

- **Users**, who read it as a reference for Cypress usage and for CI configuration across many providers.
- **The Cypress monorepo**, [cypress-io/cypress](https://github.com/cypress-io/cypress), which consumes this repository directly. See below — this is the part that is easy to break without noticing.

`app/` and `cypress/` are the source of truth for the example content. The copies inside `@packages/example` in the Cypress monorepo are generated from here, so changes start in this repository.

## How the Cypress monorepo depends on this repository

Two independent paths, both of which can break from a change here:

**1. The published npm package.** `cypress-example-kitchensink` is published from this repository by semantic-release. `@packages/example` pins it as a devDependency and copies `cypress/` and `app/` into the Cypress binary and into example.cypress.io. If publishing stops working, the Cypress release process quietly stalls the next time someone tries to bump it.

**2. A direct git clone during Cypress CI.** Cypress's pipeline clones this repository and runs it against a freshly built binary, in `test-kitchensink`, `test-kitchensink-against-staging`, and `test-binary-against-kitchensink` with Electron, Chrome, and Firefox variants. The definitions live in `.circleci/src/pipeline/@pipeline.yml` in that repository.

That second path invokes this repository's files by name, which makes them a contract:

- **npm scripts.** The pipeline runs `npm run build --if-present`, `npm run types --if-present`, `npm start --if-present`, and `npm run e2e`. There is deliberately no `types` script — adding one opts this repository into a type check it is not set up for. `test-kitchensink` additionally runs `yarn --production --ignore-engines` and `yarn start`.
- **`.node-version`.** Cypress reads it with `nvm install $(cat .node-version)`, so it has to stay a string nvm accepts.
- **npm, not yarn.** Cypress's install step branches on whether `yarn.lock` exists. Adding one silently switches it from `npm install --legacy-peer-deps` to `yarn --frozen-lockfile` plus `yarn add -D`, changing how the binary under test gets installed.
- **Branch names.** Cypress checks out `release/<next-cypress-version>` if such a branch exists here and falls back to `master`, so `master` is usually what its CI tests.

## Linting

```bash
npm run lint       # eslint
npm run lint:fix   # eslint --fix
```

`npm run build` is an alias for `npm run lint` — there is no compile step. The `pre-commit` hook runs it too.

ESLint covers more than JavaScript. `eslint.config.mjs` registers separate languages for `**/*.{,m}js`, `**/*.json` (excluding `package-lock.json`), `**/*.yml`, and `app/**/*.html`.

**The consequence worth remembering:** `.github/workflows/*.yml` is linted, `npm run build` runs that lint, and Cypress's binary jobs run `npm run build` inside their clone of this repository. A YAML style violation in a workflow file therefore fails CI in the Cypress monorepo, not only here. Run `npm run lint` before pushing workflow changes.

The YAML rules are `yml/indent` at 2, `yml/key-spacing`, `yml/no-multiple-empty-lines` capped at 1, and `yml/spaced-comment`. Note the glob is `*.yml` — a file named `.yaml` is not linted at all.

## CI

Tests run in GitHub Actions and record to Cypress Cloud [project `4b7344`](https://cloud.cypress.io/projects/4b7344/runs):

| Workflow | Coverage |
| :--- | :--- |
| `using-action.yml` | `cypress-io/github-action`, Electron, one job plus parallel 4x |
| `browsers.yml` | Chrome and Firefox on Linux (load balanced) and Windows |
| `parallel.yml` | Electron, parallel 2x, caching demonstrated by hand |
| `single.yml` | Electron, one container |
| `chrome.yml`, `chrome-docker.yml` | Non-recorded Chrome examples, the second inside a Cypress Docker image |

Things to know before editing them:

- **Triggers are `push` on `master` plus `pull_request`.** A PR from a fork therefore runs here, in this repository's Actions, and its checks appear on the pull request. Triggering only on `push` would run them in the contributor's fork instead, where the upstream pull request never sees them.
- **Recording jobs need the fork guard**, which has to cover both events:

  ```yaml
  if: github.repository_owner == 'cypress-io' && (github.event_name != 'pull_request' || github.event.pull_request.head.repo.full_name == github.repository)
  ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cypress-io/cypress-example-kitchensink](https://github.com/cypress-io/cypress-example-kitchensink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
