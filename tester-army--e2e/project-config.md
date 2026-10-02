---
trigger: always_on
description: `e2e` — an agentic end-to-end testing framework. pnpm monorepo, ESM only,
---

# AGENTS.md

`e2e` — an agentic end-to-end testing framework. pnpm monorepo, ESM only,
TypeScript 7.

## Contracts

There is no separate spec. The code is the contract, pinned in three places:

- The emitted `packages/e2e/dist/index.d.ts` (and `dist/engine/index.d.ts`,
  `dist/oauth/*.d.ts`)
  is the public API. `packages/e2e/tests/types/sdk-types.ts` holds compile-time
  assertions (`@ts-expect-error` lines) for the parts that are easy to loosen
  by accident; it runs under the package `typecheck`, never under vitest.
- Wire formats live in `packages/e2e/schema/*.schema.json` (report-1,
  session-1, agent-judgment-2, and the deprecated agent-judgment-1) with a valid and an invalid fixture
  each. Integration tests validate every generated report and session envelope
  against them; `tests/unit/schema-fixtures.test.ts` checks the fixtures. A
  wire change edits the schema, both fixtures, and the producer in one review.
- Security invariants are the list under Gotchas below, enforced by tests in
  `tests/integration/agent-policy.test.ts` and the secret-ledger unit tests.
- Behavior changes update the matching `docs/**/*.mdx` page in the same
  change, including "not implemented yet" callouts, and `skills/e2e/` when the
  changed surface is described there. `scripts/check-error-codes.ts` (in
  `pnpm check`) fails when an error code in source is missing from
  `reference/errors.mdx` or `reference/engine.mdx`, or documented but raised
  nowhere.

There are no RFCs or design documents in the repo. The why lives in PR
descriptions and commit bodies; `git log` and `gh pr view` are the archive.

## Layout

`packages/` holds what publishes to npm; `apps/` holds the private apps and
suites that consume the built packages the way a user would.

- `packages/e2e` — the published `e2e` package: SDK surface, runner, CLI,
  `e2e/engine` contract. Core knows the contract and never an engine's
  internals: no `Web`, `browser`, `page`, `route`, or `playwright` noun lives in
  `src/` (grep for them; zero hits is the invariant). The one exception is the
  `e2e init` scaffold presets in `src/cli/init/engines.ts`, which write the
  user's config and so name engine packages as text; the package build records
  the sibling packages' versions in `dist/cli/init/sibling-versions.json` for the
  ranges they write. Each preset owns its prompt label, dependencies, config,
  example, and run command; interactive choices derive from this list. These
  presets never import engine implementations.
  - `src/run/` runner core (scheduler, units, workers, retries, sessions;
    `standalone.ts` opens one attempt with no test body for hosts),
    `src/collect/` registration+selection, `src/locator/` locator AST/engine,
    `src/agent/` the agent (the `act` executor socket plus the judgment
    methods), `src/mcp/` the `e2e mcp` server (a live session that rides
    the `act` socket with a queue executor so every MCP call is a harness
    action), `src/oauth/` subscription sign-in (the `e2e login`, `logout`,
    and `models` commands and the `e2e/oauth/*` model constructors; each
    constructor subpath is the only place its `@ai-sdk/*` optional peer is
    imported, so the CLI boots without them). The constructors and the CLI
    are the whole public surface: the flows, stores, and fetch behind them
    are module-private, not a library for other products. `tests/live/` holds hand-run
    checks that need a stored login and are never part of `pnpm test`.
- `packages/web` — the published `@e2e-dev/web` package: the
  browser engine, built with the public `defineEngine`, contributing the
  `browser` fixture and `expect(browser)`. It depends on `e2e` (peer), never the
  reverse; a target names it explicitly as `engine: web()`. There is
  no default engine and no well-known id registry in core. It imports from
  `e2e/engine` only: the semantics every engine must reproduce
  (error taxonomy, text and URL matching, assertion polling, JSON-value rules)
  are exported there, and there is no `e2e/internal` subpath.
- `packages/kernel` - the published `@e2e-dev/kernel` package: Kernel hosted
  browsers for the web engine. An official integration with a hosted service
  is one package per service, named after it (`@e2e-dev/<service>`), with the
  vendor SDK and the engine it plugs into as peers. It implements that
  engine's provider seam (`BrowserProvider` for web, `DeviceProvider` for
  mobile) and imports the engine's types only; the engines never know it
  exists.
- `packages/eas` - the published `@e2e-dev/eas` package: EAS Simulators
  hosted iOS simulators and Android emulators for the mobile engine
  (`DeviceProvider`). Expo publishes no SDK for the sessions API, so it calls
  Expo's GraphQL API with `fetch`, and `@e2e-dev/mobile` is its only peer.
- `apps/testbed` (`@e2e-dev/testbed`, private) — dogfood project that
  consumes the **built** packages like a real user would: the playground app
  where every runner feature (sessions, routes, downloads, frames, uploads,
  serial groups, the executor seams, verdict edge cases, the reporter under
  stress, `explore`) has a deterministic test. Hard UI surfaces belong to the
  benchmarks, not here.
- `apps/web-benchmark` (`@e2e-dev/web-benchmark`, private) — a Next.js app of

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tester-army/e2e](https://github.com/tester-army/e2e) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
