---
trigger: always_on
description: Bun monorepo containing TypeScript client libraries for the [Seismic blockchain](https://seismic.systems) — a privacy-focused EVM chain that encrypts transaction calldata using AEAD (AES-GCM). The two published npm packages are **seismic-viem** (viem extensions) and **seismic-react** (wagmi-compatible React hooks).
---

# Seismic Client

Bun monorepo containing TypeScript client libraries for the [Seismic blockchain](https://seismic.systems) — a privacy-focused EVM chain that encrypts transaction calldata using AEAD (AES-GCM). The two published npm packages are **seismic-viem** (viem extensions) and **seismic-react** (wagmi-compatible React hooks).

## Build

Requires **Bun >=1.2.5**.

```bash
# If bun is not installed already:
curl -fsSL https://bun.sh/install | bash

bun install         # install dependencies
bun run all:build   # builds seismic-viem, seismic-react, seismic-viem-tests
```

### Verify

```bash
ls packages/seismic-viem/dist/_cjs/index.js packages/seismic-react/dist/_esm/index.js
# Both files should exist
```

### Individual package builds

```bash
bun run viem:build       # packages/seismic-viem  (CJS + ESM + types)
bun run react:build      # packages/seismic-react (CJS + ESM + types)
bun run tests:build      # packages/seismic-viem-tests (ESM + types)
```

## Checks

```bash
bun run check            # typecheck + lint (runs everything below)
bun run typecheck        # tsc --noEmit on seismic-viem then seismic-react
bun run lint:check       # eslint + prettier (no auto-fix)
bun run lint             # eslint + prettier (with auto-fix)
```

Note: `react:typecheck` rebuilds seismic-viem first (it depends on the built types).

## Tests

### Integration tests (require external tooling)

Tests live in `tests/seismic-viem/` and require a running Seismic node. Two modes:

```bash
bun run viem:test:anvil  # CHAIN=anvil — needs sanvil (from sfoundryup)
bun run viem:test:reth   # CHAIN=reth  — needs seismic-reth binary
```

**Anvil tests** require `SFOUNDRY_ROOT` env var pointing to a [seismic-foundry](https://github.com/SeismicSystems/seismic-foundry) checkout (with Rust/Cargo installed), OR the `sanvil` binary in PATH. If `SFOUNDRY_ROOT` is set, the test harness builds sanvil from source via `cargo build --bin sanvil`.

**Reth tests** require `SRETH_ROOT` pointing to a [seismic-reth](https://github.com/SeismicSystems/seismic-reth) checkout.

### Solidity tests (Foundry)

```bash
git submodule update --init --recursive   # first time only
sforge test --root contracts
```

Requires `sforge` (installed via [sfoundryup](https://github.com/SeismicSystems/seismic-foundry)). Runs 16 tests across 3 test suites (SeismicCounter, TransparentCounter, ShieldedDelegationAccount).

### Docs

```bash
bun run docs:dev         # local dev server
bun run docs:build       # production build (VoCs)
bun run docs:preview     # preview production build
```

## Project Layout

```
packages/
  seismic-viem/          Core viem extensions (npm: seismic-viem@1.1.1)
    src/
      chain.ts           Chain definitions (sanvil, seismicTestnet, localSeismicDevnet)
      client.ts          createShieldedPublicClient, createShieldedWalletClient
      sendTransaction.ts Seismic transaction sending
      contract/          getShieldedContract, shieldedWriteContract, signedReadContract
      crypto/            AES-GCM, ECDH, HKDF, nonce generation, AEAD
      precompiles/       On-chain precompile wrappers (rng, ecdh, hkdf, aes, secp256k1)
      actions/           Deposit contract, SRC20 token support
      abis/              SRC20, deposit contract, directory ABIs
  seismic-react/         React hooks (npm: seismic-react@1.1.1)
    src/
      context/           ShieldedWalletProvider
      hooks/             useShieldedWriteContract, useSignedReadContract, useShieldedContract
      rainbowkit/        RainbowKit integration
  seismic-viem-tests/    Shared test utilities (npm: seismic-viem-tests@0.1.4)
    src/
      process/           Node process management (anvil, reth spawn/kill)
      tests/             Reusable test functions (contract, precompiles, encoding, etc.)
  seismic-bot/           Slack bot for faucet management (internal)
  seismic-spammer/       Transaction load testing tool (internal)
tests/
  seismic-viem/          Integration test runner (bun test)
contracts/               Solidity contracts (Foundry)
  src/                   SeismicCounter, TransparentCounter, SRC20, DepositContract
  test/                  Foundry tests (.t.sol)
  lib/                   Git submodules (forge-std, openzeppelin, solady)
docs/                    Documentation site (VoCs + Tailwind)
```

## Code Style

**Prettier** (`.prettierrc.json`): 2-space indent, single quotes, no semicolons, 80-char width, trailing commas (ES5). Import sorting via `@trivago/prettier-plugin-sort-imports` with groups: types → external → relative (separated by blank lines).

**ESLint** (`eslint.config.cjs`): No relative import paths (`no-relative-import-paths` plugin), no unused imports. TypeScript-aware via `@typescript-eslint` with project service.

**TypeScript** (`tsconfig.base.json`, `tsconfig.json`): Shared base config with strict mode, NodeNext module resolution, ES2021 target. Each package extends the base and defines its own path aliases.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SeismicSystems/seismic-client](https://github.com/SeismicSystems/seismic-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
