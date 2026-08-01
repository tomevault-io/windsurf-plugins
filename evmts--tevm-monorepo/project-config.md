---
trigger: always_on
description: Tevm is an in-browser & Node.js-compatible Ethereum Virtual Machine (EVM) environment. It provides a complete Ethereum execution environment powered by JavaScript that can run:
---

# Tevm Monorepo Commands and Style Guide

## Project Overview

Tevm is an in-browser & Node.js-compatible Ethereum Virtual Machine (EVM) environment. It provides a complete Ethereum execution environment powered by JavaScript that can run:

**Unique Features:**

- Import Solidity contracts directly in JavaScript/TypeScript
- Typecheck Solidity contracts with full type safety
- Simulate and debug EVM execution step-by-step
- Fork mainnet or any EVM chain locally in the browser

- **In Node.js** for local development and testing
- **In the Browser** for advanced user experiences (offline simulation, real-time testing)
- **In Deno, Bun**, or any modern JavaScript runtime

### Forking Implementation

Most of Tevm's forking logic is implemented in the `StateManager`. When a fork configuration is provided:

- The StateManager creates a proxy that intercepts data requests
- If data exists locally, it returns the local version
- If data doesn't exist locally, it fetches from the remote provider
- Fetched data is cached locally for future requests
- All state changes are applied only to the local state

This enables efficient forking with minimal memory usage, as only the accessed state is loaded and stored.

### JSON-RPC Support

The entire JSON-RPC API is implemented in the `tevm/actions` package, which provides:

- Full support for standard Ethereum JSON-RPC methods (eth\_\*)
- Support for Anvil's methods (anvil\_\*)
- Debug namespace methods (debug\_\*)
- Custom Tevm methods (tevm\_\*)

These action handlers translate between Viem-style parameters and the internal Ethereumjs API, handling format conversions automatically.

## Commands

- Build: `bun build` or `nx run-many --targets=build:dist,build:app,build:types`
- Lint: `bun lint` or `biome check . --write --unsafe && biome format . --write`
- Test all: `bun test` or `bun test:run`
- Test single file: `vitest run <path-to-file>` (e.g. `vitest run packages/state/src/actions/saveStateRoot.spec.ts`)
- Test specific test: `vitest run <path-to-file> -t "<test-name>"`
- Test with coverage: `bun test:coverage`



## Style Guide

- Formatting: Biome with tabs (2 spaces wide), 120 char line width, single quotes
- Types: JavaScript with JSDoc preferred over TypeScript
- Imports: Organized by Biome, use barrel files (index.js/ts) for exports
- Naming: camelCase for functions/variables, PascalCase for classes/types
- Error handling: Extend BaseError, include detailed diagnostics
- Barrel files: Use explicit exports to prevent breaking changes


## Setup

- Package manager: pnpm 9.x.x
- Script runner: Bun
- Requires env vars for tests: TEVM_RPC_URLS_MAINNET, TEVM_RPC_URLS_OPTIMISM

## Nx can get slow

Sometimes you may notice nx slowing down single tasks taking a very long time consistentally. When this happens try running `pnpm nx reset` and it should reset it to being fast again.

## Monorepo

Tevm is a monorepo using

- [nx](https://nx.dev/concepts/mental-model) for caching and task management
- [bun](https://bun.sh/docs) for script runner and workspace node_module management
- [changesets](./.changeset/) for package versioning

For a list of all packages, see the `workspaces` key in the root level [package.json](./package.json)

## Packages

Tevm is heavily broken up into small packages. To see the entire package graph use nx

```bash
bun run nx graph
```

For more information the [reference docs](https://tevm.sh) are a useful resource. They are generated from the source code and link back to it

#### Build packages

- [@tevm/ts-plugin](./ts-plugin) is the LSP (language service protocol) plugin. It is what allows code editors such as VSCode to correctly infer types of solidity imports.
- [bundlers/\*](./bundlers) Are where all the supported bundlers live. Most are created using [@tevm/unplugin](./bundlers/unplugin) which implements a rollup plugin once to be reused in many packages including [@tevm/webpack](./bundlers/webpack) and [@tevm/vite](./bundlers/vite) and more.
- [@tevm/config](./config) is the package that loads the Tevm config

#### Runtime packages

- [@tevm/contract](./core) is the main entry point for all runtime functionality.

#### Docs

- [@tevm/docs](./docs) is the [vitepress docs site](https://tevm.sh). Its reference docs are generated via `bun generate:docs`

#### Example apps

Example apps are in [/examples/\*](./examples). [@tevm/example-esbuild](./examples/esbuild) has a vitest test and is the simplist app.

## Code best practices

#### JavaScript with jsdoc

`@tevm` is written in javascript with jsdoc so its `esm` build does not need to be built. This means any user using modern `esm` will be using the same src code in their node_modules as what is here.

This means all tevm packages run without being built by default and the same src code is shipped to most users

- [src/index.js](./src/index.js) - the entrypoint to the package

**Note** there is still 100% typesafety. TypeScript is able to typecheck via setting `checkJs: true` in the tsconfig and using jsdoc.

#### Import preferences

- Prefer inline imports in JSDoc comments where the type is used instead of global imports at the top of the file

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [evmts/tevm-monorepo](https://github.com/evmts/tevm-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
