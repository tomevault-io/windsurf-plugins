---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

!! Never run build on frontend locally for testing purposes !!

## Secrets

Use the 1Password Environment `loyal-frontend-devnet` for local frontend devnet secrets. The environment is mounted at `.env.1password`, which is ignored by Git through `.env*`.

Do not write plaintext secrets to `.env` files, source files, command arguments, logs, or chat. When a command needs environment variables, use the 1Password CLI with the mounted env file:

```sh
op run --env-file=.env.1password -- sh -c '<command>'
```

Keep shell variable expansion inside the `sh -c` subprocess so `op run` injects values before the command reads them.

## Project Overview

Solana Telegram Transactions enables users to deposit SOL for any Telegram username, which can later be claimed by the verified account owner. It integrates Solana smart contracts with a Telegram mini-app interface.

## Commands

### Telegram Mini-App Frontend (run from `/app`)

```bash
bun dev                    # Run mini-app dev server (turbopack)
bun run build              # Production build (Next.js)
bun lint                   # ESLint
bun db:generate            # Generate Drizzle migrations from schema
bun db:migrate             # Apply migrations
bun db:studio              # Open Drizzle Studio GUI
```

### Loyal Web Frontend (run from `/frontend`)

```bash
bun dev                    # Run Loyal web dev server (turbopack)
bun run build              # Production build (Next.js)
bun run lint               # Next.js lint
bun run ultracite          # Biome/Ultracite checks
```

### Mobile App (run from `/mobile`)

```bash
npx expo start --clear     # Start Expo dev server (requires dev client)
npx expo lint              # ESLint
npm test                   # Jest tests
npx eas build --profile development-simulator --platform ios  # Build dev client (iOS sim)
npx eas build --profile development --platform ios            # Build dev client (device)
npx eas build --profile preview --platform android            # Preview APK
npx eas build --profile production --platform all             # Production build
```

### Admin Dashboard (run from `/admin`)

```bash
bun dev                    # Run admin dev server (turbopack)
bun run build              # Production build (Next.js)
bun lint                   # Next.js lint
```

### Smart Contracts (run from root)

```bash
anchor build               # Compile programs
anchor deploy --provider.cluster devnet     # Deploy to devnet
anchor deploy --provider.cluster localnet   # Deploy to localnet
```

### Testing Smart Contracts

Requires 3 terminals running simultaneously:

```bash
# Terminal 1: Start validator
mb-test-validator --reset

# Terminal 2: Start ephemeral validator
RUST_LOG=info ephemeral-validator \
    --accounts-lifecycle ephemeral \
    --remote-cluster development \
    --remote-url http://127.0.0.1:8899 \
    --remote-ws-url ws://127.0.0.1:8900 \
    --rpc-port 7799

# Terminal 3: Run tests
EPHEMERAL_PROVIDER_ENDPOINT="http://localhost:7799" \
EPHEMERAL_WS_ENDPOINT="ws://localhost:7800" \
anchor test --provider.cluster localnet --skip-local-validator --skip-build --skip-deploy
```

## Test Design Guardrails

Default to no new TypeScript unit tests. Prefer typecheck, lint, focused
verifier scripts, manual smoke checks, or live read-only probes over `bun:test`,
Jest, Vitest, `*.test.ts(x)`, `*.spec.ts(x)`, or `__tests__/` coverage.

Keep or add a TypeScript test only when it protects an external contract or
invariant that would still compile while broken: money movement,
auth/session boundaries, confirmed-chain writes, generated SDK or wire-format
parity, public API discriminants, webhook retry behavior, DB
ownership/idempotency/conflict behavior, signer/secret/storage boundaries,
config parsing, or dangerous pure calculations.

Implement the core logic requested first. Only add or update tests after the
behavior exists, and only when the test still passes this rubric:

A test earns `+1` when it protects an external contract or invariant static
checks can miss, `+1` when it asserts an observable side effect or invariant,
and `+1` when TypeScript, lint, or a manual smoke check would not catch the
failure. Observable checks include write/no-write behavior, ordering, rejection
before mutation, retry/backoff, persisted state transitions, balance or amount
calculation, and instruction/account ordering.

Subtract `1` when the test mostly mirrors fields, defaults, copy, route
strings, source strings, `Object.keys(...)`, `typeof ...`, or third-party
library behavior. Subtract `1` when it mainly asserts mocked JSON, Drizzle call
order, every `.values()` field, a full response body, or that a mocked
dependency received the same mock data.

Keep only tests scoring at least `2`. For mixed files, delete or compress the
low-scoring blocks and keep only the contract/invariant checks.

Delete or compress tests that restate implementation shape. Do not keep tests
whose main value is asserting every mocked JSON field, every Drizzle `.values()`
field, object defaults, route-string builders, source substrings, copy that is
not a product or ops contract, `typeof ... === "function"`, `Object.keys(...)`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [loyal-labs/loyal-app](https://github.com/loyal-labs/loyal-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
