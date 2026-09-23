---
trigger: always_on
description: Read `README.md` before changing the application. Preserve the backend-neutral
---

# Repository agent guide

Read `README.md` before changing the application. Preserve the backend-neutral
contracts: Pi-specific SDK types, event parsing, and history interpretation stay
under `src/server/backends/pi`; the browser consumes only normalized protocol
types.

## Changelog and releases

Record user- and operator-visible changes in `CHANGELOG.md` under `Unreleased`.
Use Breaking Changes, Added, Changed, Fixed, and Removed as needed; do not
create duplicate subsections. Keep notes concise, explain compatibility or
upgrade effects, and add the PR number or link after opening the PR and before
merging. The initial release entry is only `Initial release`; do not restore a
catalogue of pre-release development history.

Follow [Release process](docs/developer/release-process.md). Use
`npm run release:prepare -- X.Y.Z` (or `patch`, `minor`, `major`) on a clean
branch to synchronize product versions and roll the changelog for review.
Use `--dry-run` to preview. Commit and merge the prepared release normally.
Publish only when requested, using `npm run release:publish -- X.Y.Z` from
clean `main` matching origin. This creates an annotated tag and a GitHub
release containing changelog notes. Current releases are source-only; there
is no npm publication, binary upload, or deployment step.

Keep product versions synchronized with `scripts/version.mjs`, even though
packages are private. Browser and provider protocol versions are independent
compatibility contracts; a product release alone does not bump them.

## Backend-facing changes

Before adding a backend or changing a backend-facing contract or cross-cutting
feature, read and follow `docs/internals/backend-integration-contract-rules.md`. Update
that guide when the change adds a reusable integration rule, invariant, or
required audit surface. Audit every compiled backend and give each an explicit
implemented or intentionally unsupported disposition through truthful
capabilities. Implement and test every required backend path, including
unsupported and fail-closed behavior. Keep provider protocols, native
identifiers, event/history interpretation, transports, and topology private to
their backend; keep browser contracts normalized. Remove obsolete shapes
instead of adding silent aliases, fallback parsers, bridge routes, or dual
contracts.

## Ownership and tenancy

Before adding a feature, classify its configuration, persisted state, events,
side effects, and controls by their intended ownership boundary: installation
system, tenant, principal, execution environment, workspace, or thread. Keep
operator-owned system configuration distinct from tenant/principal-owned
application state, and document explicit precedence and mutability when more
than one scope can contribute policy.

The current production identity provider intentionally supports exactly one
local principal. Treat that as a product limitation, not as permission to make
principal-owned state globally scoped. Build repositories, service authority,
receipts, queues, event dispatch, caches, and runtime keys around the
server-derived `tenantId`/`principalId` boundary and test wrong-scope denial,
while exposing only the single-principal UI that is truthfully supported now.
Never accept a browser-selected tenant or principal as authority, simulate
tenant administration before authenticated roles exist, or allow one scope's
state to become another scope's fallback.

Use Node.js 24.18 or newer and install with `npm ci`. The standard verification
sequence is:

```sh
npm run typecheck
npm test
npm run build
npm run test:e2e
```

Run installs, tests, and builds with `NODE_ENV` unset. A shell-exported
`NODE_ENV=production` makes `npm ci` silently prune devDependencies
(breaking `vitest.config.ts` and Playwright) and makes Vitest resolve
React's production build, which fails most client component tests with
`React.act is not a function`. If client tests fail that way, reinstall
with `env -u NODE_ENV npm ci` and rerun the sequence with `env -u NODE_ENV`
prefixed to each command.

The E2E coordinator uses invocation-scoped disposable state and writes
screenshots beneath the run directory it prints, normally at
`test-results/e2e-runs/run-*/jobs/*/screenshots/` and at
`test-results/e2e-runs/run-*/screenshots/` for single-lane runs. Inspect changed
screenshots, including an in-flight streaming state.

Before adding or changing Playwright coverage, read and follow
[`docs/developer/e2e-testing.md`](docs/developer/e2e-testing.md). Keep every spec independently
runnable; never depend on another spec's warmed process, state, workspace, or
execution order. Treat each spec file as one indivisible scheduled job, reserve
serial suites for genuine same-server state chains, and split long independent
chains without creating tiny files solely for parallelism. On this reference
host, target 210 seconds for the full build-inclusive suite and 200 seconds for
the parallel prebuilt suite under the stable default four-lane schedule. These
are regression signals, not acceptance
gates or portable test timeouts. Never weaken coverage or assertions, split a
genuine same-server state chain, introduce shared state, or use an unrealistic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kcosr/sedes](https://github.com/kcosr/sedes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
