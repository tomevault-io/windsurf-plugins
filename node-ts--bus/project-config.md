---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

`@node-ts/bus` is a pnpm monorepo for a TypeScript service bus: message handlers, workflows (sagas), retries, and pluggable transports/persistence. Consumer docs live at https://node-ts.github.io/bus and are built from `docs/` (see Docs under Conventions).

Requires Node.js 24 or later: every published package declares `engines.node >=24`, and the tsconfig base is `@tsconfig/node24`.

## Commands

pnpm only (enforced by `preinstall`). The versions are pinned to pnpm 12.4.1 (`packageManager`) and node 24.11.1 (`.nvmrc`); CircleCI uses the same versions. pnpm 12 fails the install if a dependency's build script hasn't been approved or denied in `allowBuilds` in `pnpm-workspace.yaml`. When you add a dependency that has a build script, add it there.

```sh
pnpm i
pnpm build                 # tsc in every package (outputs to each package's dist/)
pnpm build:watch
pnpm check:packages        # after build: pack each package, run publint + attw, check ESM and CJS entries match
pnpm check:message-types   # after build: fail if a committed message-types.generated.ts is out of date
pnpm format                # prettier write; CI runs `pnpm format:check`
pnpm lint                  # after build: ESLint (eslint.config.mjs, type-aware typescript-eslint); CI runs it too
pnpm test                  # all spec + integration tests, with coverage
pnpm test:unit             # *.spec.ts only
pnpm test:integration      # every package's *.integration.ts (runInBand, needs the infra below)

# Single file / single test (always go through dotenv so test.env is loaded)
pnpm exec dotenv -e test.env -- jest packages/bus-core/src/service-bus/bus-instance.integration.ts
pnpm exec dotenv -e test.env -- jest packages/bus-sqs/src/sqs-transport.spec.ts -t "some test name"
```

- Jest is configured once at the root (`jest.config.ts`, ts-jest using `tsconfig.test.json`; a `moduleNameMapper` lets relative `.js` imports, as in generated message types, resolve to `.ts` source); `test/setup.ts` drops the default logger's `@node-ts/…` console warnings and errors (set `BUS_TEST_LOGS=true` to see them). Other console output still shows, so don't leave `console.log` in tests.
- Tests: `*.spec.ts` = unit, `*.integration.ts` = integration. `packages/bus-test/src/in-memory.integration.ts` runs bus-test's own round trip suites over the in-memory queue and persistence.
- **Build before testing across packages**: every package has `main: ./dist/index.js`, so e.g. bus-sqs tests import the built bus-core and bus-test, not their source. Changes in bus-core need `pnpm build` before they're visible to other packages.
- Integration tests for adapters need local infra; `docker compose up -d` starts it all from the root `docker-compose.yml`. Endpoints default to the compose ports and can be overridden with env vars (listed in `test.env`): `LOCALSTACK_ENDPOINT` (default `http://localhost:4566`, dummy AWS creds in `test.env`), `RABBITMQ_URL` (`amqp://guest:guest@0.0.0.0`), `POSTGRES_URL` (`postgres://postgres:password@localhost:6432/postgres`), `MONGODB_URL` (`mongodb://localhost:27017/workflows`). CI runs all integration tests against CircleCI secondary containers (`.circleci/config.yml`).
- Formatting is automatic: a `.claude/settings.json` hook runs prettier on every file Claude edits, and husky + lint-staged format on commit.
- Linting: `eslint.config.mjs` is `recommended` from ESLint and typescript-eslint, plus the type-aware `no-floating-promises`, `no-misused-promises` and `await-thenable`, with `eslint-config-prettier` so it never fights prettier. Type information comes from `tsconfig.eslint.json` (specs and `test/` fixtures included), and cross-package types come from each package's `dist`, so build first. lint-staged also runs ESLint on staged TS/JS files. Await or `.catch` every promise; use `void` only for deliberate fire-and-forget, with a comment saying why. Unused parameters are allowed when prefixed with `_`.
- Dependency updates are grouped by Renovate (`renovate.json`); the maintainer installs the Renovate GitHub app.
- Docs: `pnpm docs:dev`, `pnpm docs:typecheck`, `pnpm docs:build`, `pnpm docs:check-redirects` and `pnpm docs:check-readmes` (see the Docs convention below). CircleCI's `docs` job runs the last four on every branch; `.github/workflows/docs.yml` deploys the site to GitHub Pages on each GitHub Release, or when run by hand.
- Releases use [changesets](https://changesets.dev) with linked package versions (`.changeset/config.json`, workflow in `CONTRIBUTING.md`, 1.x → 2.0 upgrade notes in `MIGRATING.md`). On master, CircleCI's deploy job runs `pnpm changeset publish` (publishes any package whose version isn't on npm yet) and then `.circleci/create-github-releases.mjs` (a GitHub Release and `<pkg>@<version>` tag per new version, with its CHANGELOG section as notes).
- Package-specific notes (design, config defaults, local infra, gotchas) live in `packages/<pkg>/CLAUDE.md`. Scaffolding new adapters is covered by the `add-transport` and `add-persistence` skills in `.claude/skills/`.

## Architecture

### Packages


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [node-ts/bus](https://github.com/node-ts/bus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
