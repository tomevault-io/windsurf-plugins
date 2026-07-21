---
trigger: always_on
description: per https://agents.md/
---

# Repository Guidelines

per https://agents.md/

## Project Structure & Module Organization
- Monorepo managed by Yarn workspaces and Lerna Lite. Primary code lives under `packages/*` (e.g., `SwingSet`, `zoe`, `ERTP`, `smart-wallet`).
- Go components are under `golang/` (e.g., `golang/cosmos`).
- Tests reside per package in `packages/<name>/test/`.
- Utilities, CI and developer tooling scripts are in `scripts/`. Integration assets live in `a3p-integration/` and `multichain-testing/`.

## Build, Test, and Development Commands
- `corepack enable && yarn install`: Set up the repo with the pinned Yarn version and install dependencies.
- `yarn build`: Build all workspaces (generates kernel bundles where needed).
- `yarn test`: Run unit tests across all packages (AVA).
- `yarn lint` | `yarn lint-fix`: Check or auto-fix lint issues across packages.
- `yarn lint:types`: Fast typecheck within a package (uses tsgo, the TypeScript 7 native preview); do this after changes.
    - Watch mode for type errors in active workspaces: run `yarn lint:types --watch --preserveWatchOutput` in the workspace(s) being edited, and keep the terminal output visible so Codex can monitor errors.
- `yarn lint:eslint`: If this script exists in `package.json`, run it in the background after changes.  It is slower but catches errors that `yarn lint:types` may not.
- `yarn typecheck-all` to do a fast typecheck over the whole repo (a few seconds, using tsgo)
- `yarn format`: Format code via dprint; `yarn lint:format` to check only.
- Git hooks: installed by `scripts/install-git-hooks.sh`.
  - Install or refresh hooks with `yarn hooks:install`.
  - Pre-commit runs `scripts/git-hooks/pre-commit-dprint.sh`, which formats staged JS/TS files with the pinned local binary `./node_modules/.bin/dprint`, auto-restages files that were fully staged already, and prints a custom message if formatting changes touched partially staged files.
- `./scripts/env-doctor.sh`: Verify toolchain (Node, Go, compiler) versions.
- Example, single package: `cd packages/eventual-send && yarn test`.
- Packing/debugging workflow:
  - Full sequential prepack pass across publishable packages: `yarn lerna run --reject-cycles --concurrency 1 prepack`
  - If a package fails, fix it and verify locally in that package with `yarn postpack && yarn prepack`
  - Resume from the failed package and include dependents needed for validation: `yarn lerna run prepack --since <failed-package-name> --include-dependencies --concurrency 1 --reject-cycles`
  - After any prepack run, clean generated artifacts and restore package trees with: `yarn lerna run --reject-cycles --concurrency 1 postpack`

## Coding Style & Naming Conventions
- ESM by default; JS and TypeScript both used. Target Node ^22.11.
- dprint enforced (Prettier-compatible options include single quotes and trailing commas).
- ESLint configured via `eslint.config.mjs` (includes AVA, TypeScript, JSDoc, and repository-specific rules).
- Package names: publishable packages use `@agoric/*`; private/local packages use `@aglocal/*` (verify with `yarn lint:package-names`).
- `@aglocal` packages are private and never published; `@agoric` packages are published and may only depend on published packages, so `@agoric` packages must never import `@aglocal` packages.
- For elapsed duration measurement (benchmarks, latency logs, monotonic timeout windows), prefer `performance.now()` over `Date.now()`. Use `Date.now()` for wall-clock timestamps, IDs, and protocol deadlines.
- Entrypoints vs modules
    - Keep ambient authority (e.g., `process.env`, `console`, filesystem, network) in entrypoints
    - pass explicit capabilities (e.g., `io.console`) into shared JS modules.
    - Never `@endo/init` in modules; best practice is at the beginning of an entrypoint

## Capability Security & POLA

This is a capability-security codebase. Apply the [Principle of Least Authority](https://docs.agoric.com/guides/js-programming/hardened-js.html#the-principle-of-least-authority-pola) by default: every object — facet, capability, callback — should hold and expose only the authority needed to do its legitimate job. POLA "limits the damage that can happen if there is an exploitable bug."

When you add a method or pass a capability, ask **"who can reach this, and what stops an unauthorized caller?"** Use the answer to decide placement:

- Per-principal operations (acting on one portfolio, account, vault, …) belong on that principal's facet, not on the public facet. Zoe exposes a contract's `publicFacet` to anyone with the instance (`E(zoe).getPublicFacet(instance)`), so a method placed there is reachable by anyone — only acceptable when it creates new state owned by a verifiable principal, returns pure info, or carries its own proof of authority.
- A dispatch lookup that throws on miss (e.g., `wallet.foo.get(id)`) is often the entire enforcement mechanism for "the caller must own this thing." Don't sidestep that pattern when adding a new op — route through the same per-principal reference.
- Pass narrowed capabilities into modules, not whole objects. Inline adapters like `{ publish: node.setValue }` are cheap; over-broad refs are expensive when they leak. The "Entrypoints vs modules" rule above is one instance of this.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Agoric/agoric-sdk](https://github.com/Agoric/agoric-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
