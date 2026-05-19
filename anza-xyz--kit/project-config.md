---
trigger: always_on
description: Kit (`@solana/kit`) is the official JavaScript SDK for building Solana applications, maintained by Anza. The official documentation lives at https://solanakit.com (docs at `/docs`, API reference at `/api`). The GitHub repo is `anza-xyz/kit`.
---

# Agent Instructions

## Project Overview

Kit (`@solana/kit`) is the official JavaScript SDK for building Solana applications, maintained by Anza. The official documentation lives at https://solanakit.com (docs at `/docs`, API reference at `/api`). The GitHub repo is `anza-xyz/kit`.

This is a pnpm monorepo with ~57 packages under `packages/`, orchestrated by Turborepo. The main `@solana/kit` package is a facade that re-exports ~20 workspace packages (accounts, addresses, codecs, errors, keys, rpc, signers, transactions, etc.) and adds its own convenience helpers like `sendAndConfirmTransaction`, `airdrop`, and `fetchLookupTables`.

The documentation website supports an `.mdx` suffix on any page (e.g. `https://solanakit.com/docs/concepts/transactions.mdx`) which returns a cleaner markdown representation suitable for LLM consumption. Use this when you need to look up Kit APIs or concepts.

There is also a companion [`anza-xyz/kit-plugins`](https://github.com/anza-xyz/kit-plugins) monorepo that builds an abstraction layer on top of Kit by offering composable plugins and ready-to-use clients (which are essentially curated sets of plugins). The plugin system relies on the `@solana/plugin-core` and `@solana/plugin-interfaces` packages defined in this repo. Additionally, `@solana/program-client-core` (re-exported from `@solana/kit/program-client-core`) provides the helpers used by the [Codama JS renderer](https://github.com/codama-idl/renderers-js) to generate Kit-compatible program clients.

## Getting Started

```shell
pnpm install                                   # Install dependencies
pnpm test:setup                                # Install the Agave test validator (first time only)
./scripts/start-shared-test-validator.sh       # Start shared test validator (needed for some tests)
pnpm build                                     # Build + test all packages
```

For single-package development:

```shell
cd packages/<name>
pnpm turbo compile:js compile:typedefs         # Compile the package and its dependencies
pnpm dev                                       # Run tests in watch mode
```

## Architecture

The packages form a layered dependency graph. `@solana/errors` is the foundational leaf package with no internal dependencies — nearly every other package depends on it. Core encoding lives in `@solana/codecs-core`, `@solana/codecs-numbers`, `@solana/codecs-strings`, and `@solana/codecs-data-structures`, unified by the `@solana/codecs` facade. `@solana/keys` and `@solana/addresses` build on these codecs and `@solana/nominal-types`. Higher-level packages like `@solana/transactions`, `@solana/rpc`, and `@solana/signers` compose these primitives. `@solana/kit` sits at the top as the consumer-facing umbrella.

Four private "impl" packages (`@solana/crypto-impl`, `@solana/text-encoding-impl`, `@solana/ws-impl`, `@solana/event-target-impl`) provide platform-specific implementations and are **inlined at build time** by tsup — they are never published to npm.

## Build System

- **JS compilation**: tsup (esbuild) via configs in `packages/build-scripts/`.
- **Type declarations**: tsc with per-package `tsconfig.declarations.json`.
- **Build artifacts per package**: Node CJS + ESM, Browser CJS + ESM, React Native ESM. `@solana/kit` additionally produces IIFE bundles (minified production + development with sourcemaps).
- **Platform constants** (set at build time): `__BROWSER__`, `__NODEJS__`, `__REACTNATIVE__`, `__VERSION__`. These are mutually exclusive booleans enabling platform-specific code paths to be tree-shaken.
- **`__DEV__`**: In IIFE builds, statically `true`/`false`. In library builds, replaced with `process.env.NODE_ENV !== 'production'` by the DevFlagPlugin, deferring the decision to the consumer's bundler.

## Testing

- **Framework**: Jest v30 with `@swc/jest` for TypeScript transformation.
- **Shared config**: `packages/test-config/` (`@solana/test-config`).
- **File naming**: `*-test.ts` (runs in both environments), `*-test.node.ts` (Node only), `*-test.browser.ts` (browser/jsdom only). Tests live in `src/__tests__/`.
- **Type tests**: `src/__typetests__/` directories contain compile-time type tests using `satisfies` and `@ts-expect-error`.
- **Lint/prettier**: Also run through Jest runners (`jest-runner-eslint`, `jest-runner-prettier`).
- **Commands**: `pnpm test` runs all unit tests. `pnpm lint` runs lint checks. `pnpm style:fix` auto-fixes formatting.
- **`expect.assertions`**: Only use `expect.assertions(n)` in **async** tests (where you need to guarantee the expected number of assertions ran). Synchronous tests do not need it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anza-xyz/kit](https://github.com/anza-xyz/kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
