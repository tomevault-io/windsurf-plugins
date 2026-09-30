---
trigger: always_on
description: Context and rules for AI agents working on `verdaccio/verdaccio`. `CLAUDE.md` is a
---

# Agent Guide to the Verdaccio Repository

Context and rules for AI agents working on `verdaccio/verdaccio`. `CLAUDE.md` is a
symlink to this file. Humans are welcome to follow it too; where it repeats
[CONTRIBUTING.md](./CONTRIBUTING.md), CONTRIBUTING.md wins.

## What this branch is

`master` is the **9.x experimental** line: the monorepo that produces every
`@verdaccio/*` internal package, the `@verdaccio/ui-theme` web UI, and the
`verdaccio` binary in `packages/verdaccio`. It publishes under the npm tag
`next-9` and the Docker tag `nightly-master`. The support table, npm tags, and
Node.js policy for every release line live in [VERSIONS.md](./VERSIONS.md); the
security policy in [SECURITY.md](./SECURITY.md).

The other branches of this repository:

| Branch   | Role                                                                                                      |
| -------- | --------------------------------------------------------------------------------------------------------- |
| `master` | 9.x experimental. Source of truth for all `@verdaccio/*` packages and the UI theme. New features go here. |
| `6.x`    | Current stable `verdaccio` (npm `latest`). Bug fixes only. Its `@verdaccio/*` internals come from `8.x`.  |
| `7.x`    | Next major binary (npm `next-7`). Its `@verdaccio/*` internals come from `master`.                        |
| `8.x`    | Internal modules consumed by 6.x. Bug fixes only, no public `verdaccio` release.                          |

Rules that follow from this layout:

- **New features land on `master` only.** A feature that exists only on 9.x is
  not a parity gap.
- **A bug fix goes to every supported line that has the bug.** Fix it on
  `master` first, then port it to `6.x` (and to `8.x` when the bug lives in an
  internal module that 6.x consumes). One PR per branch. When the port is not in
  the same batch of work, say so in the PR description.
- **Each branch has its own toolchain.** Read the target branch's `package.json`
  (`engines.node`, `packageManager`) before running anything there; `6.x` uses
  yarn, the others use pnpm, and the Node.js floors differ.
- **Security reports never go through public issues or PR discussion.** Point
  reporters to SECURITY.md. On 9.x, security findings are handled as regular
  bugs, but the fix still avoids describing the exploit in public before a
  stable release carries it.

## Repository structure

Every package lives under `packages/` and is published as `@verdaccio/<name>`
unless noted. The request path is roughly `verdaccio` → `server` → `api`/`web`
→ `store` → `local-storage` plugin / `proxy` uplinks.

- `packages/verdaccio` — the `verdaccio` binary: `bin/verdaccio`, `src/start.ts`.
- `packages/cli` — CLI commands. `packages/node-api` — programmatic `runServer`
  / `startServer`.
- `packages/server/express` — the Express application (`@verdaccio/server`).
- `packages/api` — the npm registry HTTP API: publish, dist-tags, search,
  user/login/token, stage, whoami, ping.
- `packages/web` — web UI endpoints and middleware. `packages/plugins/ui-theme`
  and `packages/ui-components` — the React UI (`packages/ui-components` has
  Storybook).
- `packages/store` — storage orchestration: local packages, uplink merge,
  filter pipeline, stage storage. `packages/plugins/local-storage` — the default
  filesystem storage plugin (`@verdaccio/local-storage`).
- `packages/proxy` — the uplink HTTP client.
- `packages/auth` — authentication, tokens, 2FA. `packages/plugins/htpasswd` and
  `packages/plugins/auth-memory` — bundled auth plugins.
- `packages/config` — configuration parsing, defaults, package access rules,
  uplinks, security settings.
- `packages/core/core` — shared utilities (`errorUtils`, `validationUtils`,
  `pkgUtils`, `searchUtils`, `streamUtils`, constants such as `HTTP_STATUS` and
  `API_ERROR`). `packages/core/types` — shared TypeScript types.
  `packages/core/tarball`, `packages/core/url`, `packages/core/file-locking`,
  `packages/core/i18n` — focused helpers.
- `packages/middleware`, `packages/loaders` (plugin loading), `packages/logger`,
  `packages/hooks` (notifications), `packages/search`, `packages/signature`.
- `packages/plugins/audit`, `packages/plugins/memory`,
  `packages/plugins/package-filter` — bundled plugins (`verdaccio-audit`,
  `verdaccio-memory`, `@verdaccio/package-filter`).
- `packages/tools/helpers` — `@verdaccio/test-helper`: `initializeServer`,
  package metadata generators, publish helpers. The other `packages/tools/*`
  are private fixtures and release tooling.
- `e2e/` — end-to-end notes and Docker flows. The CLI battery comes from
  `@verdaccio/e2e-cli` (repository `verdaccio/e2e-tests`); the UI battery is
  Cypress under `cypress/`.
- `docs/` — migration guide and warning codes. `docker-examples/` — reverse
  proxy and deployment examples.

## Setup, build, test, lint

```bash
pnpm install                       # also installs the husky hooks
pnpm build                         # every package, required before tests
pnpm test                          # every package
pnpm --filter @verdaccio/store test                                # one package
pnpm --filter @verdaccio/store test test/versions.spec.ts          # one file

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [verdaccio/verdaccio](https://github.com/verdaccio/verdaccio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
