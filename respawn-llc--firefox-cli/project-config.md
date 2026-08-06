---
trigger: always_on
description: `firefox-cli` is a Bun/TypeScript workspace for controlling a user's normal Firefox session from a terminal. It exposes an npm-distributed CLI named `firefox-cli`, a native messaging host, a Firefox WebExtension, and a shared protocol package.
---

## Context

`firefox-cli` is a Bun/TypeScript workspace for controlling a user's normal Firefox session from a terminal. It exposes an npm-distributed CLI named `firefox-cli`, a native messaging host, a Firefox WebExtension, and a shared protocol package.

The CLI must talk to the user's real Firefox session through the extension and native host. Do not base product behavior on launching a separate automation-only browser profile.

## Commands / Workflow Guidance

Use the root Bun workspace scripts as the source of truth; CI runs the same checks. The workspace is pinned to Bun through `packageManager` in `package.json` and uses `bun.lock` as the only dependency lockfile.

- Run `bun run check` for the root quality gate. It runs format checking, version sync checking, dependency and TypeScript policy gates, Biome, ESLint, source-size checks, typecheck, unit tests, build, extension lint, and package layout checks.
- Run `bun run typecheck` after cross-package type or protocol changes when a full check is not needed yet.
- Run `bun run test` for unit and policy tests.
- Run `bun run test:e2e` for native-host/package smoke coverage. Disposable Firefox E2E launches only when `FIREFOX_CLI_E2E_DISPOSABLE=1` is set.
- Run `bun run extension:build` to create the loadable development extension in `dist/extension`.
- Run `bun run package:check` after packaging or install-layout changes.
- Run `bun run release:check`, `bun run release:check:local`, or `bun run release:check:signed` for release-package verification depending on whether a signed XPI is required.
- Run `bun run deps:check` when dependency manifests or lockfiles change.

Package-local scripts exist for focused `format`, `lint`, `test`, and `typecheck`, but include a root typecheck or stronger root check before finishing work that touches package boundaries.

## Testing Instructions

Write or update tests with behavior changes in protocol schemas, CLI command parsing/output, native-host transport/pairing, extension command handlers, browser API adapters, and packaging/release scripts.

Test typed request/response contracts and stable error mapping rather than duplicating implementation details. Keep browser-facing behavior testable without a live Firefox profile by isolating WebExtension APIs behind small adapters and mocks.

Add integration or smoke coverage for CLI-to-extension transport, native messaging setup, local IPC, pairing state, install layout, binary output paths, and packaging changes. These paths cross process and browser boundaries and are not protected by TypeScript alone.

Use Vitest for workspace TypeScript tests and Node's built-in test runner for policy tests under `scripts/check-*.test.mjs`. Keep security-sensitive native-host behavior covered by targeted tests.

## Project Layout & Module Map

Use package boundaries that preserve the CLI, native host, extension, and shared protocol:

- `packages/cli`: CLI entrypoint, argument parsing, terminal output, setup/doctor UX, local IPC client, and packaging entrypoint.
- `packages/extension`: Firefox MV3 WebExtension source, manifest, popup, browser API adapters, background/content scripts, permissions, and extension packaging assets.
- `packages/native-host`: Firefox native messaging stdio handling, per-user local IPC, IPC auth, native-host manifest registration primitives, pairing state, platform binary resolution, file writes for binary outputs, and extension connection brokering.
- `packages/protocol`: command names, request/response schemas, runtime validation, protocol versioning, and compatibility helpers shared by the CLI and extension.
- `packages/test-support`: fixtures, fake transports, browser API mocks, and cross-package test helpers.
- `scripts`: repository automation for package assembly, extension bundling/signing, native-messaging manifest checks, release verification, dependency policy, and E2E smoke workflows.
- `docs`: user-facing setup, command reference, development, architecture, and capability notes.

Import workspace packages through their public package exports. Do not deep-import another package's `src` internals.

## Dev Environment Tips

- Keep developer-specific Firefox profile paths, generated extension IDs, native-host locations, pair tokens, approval state, and local install state out of tracked files.
- Do not write tests that use a real Firefox profile or real user native-messaging manifest locations by default. Use temporary paths unless a setup command explicitly requests a real install mutation.
- Do not kill, restart, or mutate a real user Firefox process during development or QA automation.
- Do not manually copy generated artifacts between packages. Build, package, native-host registration, and doctor flows must be executable from root scripts or `firefox-cli setup`/`firefox-cli doctor`.
- Update `dependency-policy.json` when adding or removing direct dependencies. The policy gate rejects unreviewed direct dependencies and non-Bun lockfiles.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [respawn-llc/firefox-cli](https://github.com/respawn-llc/firefox-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
