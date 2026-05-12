---
trigger: always_on
description: Instructions for AI coding agents working on the **Actions SDK monorepo itself**. If you are an agent helping a developer *integrate* the published SDK into their app, see [`llms-full.txt`](./llms-full.txt) instead.
---

# AGENTS.md

Instructions for AI coding agents working on the **Actions SDK monorepo itself**. If you are an agent helping a developer *integrate* the published SDK into their app, see [`llms-full.txt`](./llms-full.txt) instead.

This file is the actionable rules. The full prose, rationale, and examples live in [CONTRIBUTING.md](./CONTRIBUTING.md). When in doubt, read CONTRIBUTING.md; it is canonical.

## Repo at a glance

- **Monorepo**: pnpm workspaces. Packages in `packages/`.
  - `packages/sdk/`: the published TypeScript SDK (`@eth-optimism/actions-sdk`)
  - `packages/demo/backend/`: Hono-based Node.js demo
  - `packages/demo/frontend/`: React + Vite demo
- **Node**: >= 18.
- **SDK shape**: provider pattern. Domains live under `packages/sdk/src/actions/<domain>/` with a `core/` base class, `providers/<protocol>/` concrete implementations, `namespaces/`, `__mocks__/`. Cross-environment factories in `nodeActionsFactory.ts` / `reactActionsFactory.ts`. Built on viem.
- **Today's domains**: `lend/`, `swap/`. `borrow/` is anticipated; new domains follow the same shape.

## Verify before commit

```bash
pnpm typecheck && pnpm lint && pnpm test
```

Zero new lint warnings, zero new TypeScript errors. The baseline only goes down.

## Hard rules

The bullets below are the rules. Each links to the section in CONTRIBUTING.md with the *why*.

### [Testing](./CONTRIBUTING.md#testing)

- Every new feature ships with tests. PR doesn't merge until CI is green.
- Unit tests for logic; integration tests for cross-layer behavior. Most non-trivial changes need both.
- Mock at boundaries (RPC, wallet provider, bundler, third-party SDKs). **Never mock `encodeFunctionData`, `decodeFunctionData`, `keccak256`, `encodeAbiParameters`.**
- Bug fixes require a regression test that fails without the fix.

### [Reuse before invention](./CONTRIBUTING.md#reuse-before-invention)

- Grep before writing any new utility, mock, fixture, or helper.
- First places to check: `packages/sdk/src/__mocks__/`, per-domain `__mocks__/`, `packages/sdk/src/utils/{validation,approve,assets}.ts`, `packages/demo/backend/src/utils/serializers.ts`, `packages/demo/frontend/src/utils/formatters.ts`.
- Naming: `*Raw` suffix is reserved for `bigint`; verbs are `getX` / `buildX` / `resolveX` / `validateX`.

### [Prefer viem patterns](./CONTRIBUTING.md#prefer-patterns-set-by-direct-dependencies-especially-viem)

- Errors: named concrete classes extending `BaseError`. Discriminate via `instanceof`, not `error.code`.
- ABIs: `as const`. Don't widen to `Abi`.
- Encoding: `encodeFunctionData` / `decodeFunctionData` with the ABI inline.
- Clients: multicall and transport composition over hand-rolled batchers.
- **Extraction trigger: the second concrete usage, not a speculative first.**

### [Type safety](./CONTRIBUTING.md#type-safety-and-precision)

- No `any`. Use `unknown` and narrow.
- No unsafe casts (`as Foo`); tighten the upstream type instead.
- Prefer the narrowest accurate type: `SupportedChainId` over `number`, `Hex`/`Address` over `string`.
- Tighten loose types in the same PR you touch them. The baseline ratchets down.
- `readonly T[]` for non-mutated collection params. Discriminated unions over both-optional-one-required. `import type` for type-only imports.

### [Abstraction hierarchy](./CONTRIBUTING.md#abstraction-hierarchy)

- Concrete providers stay thin. Validation, amount conversion, approval building, calldata verification, error wrapping, and chain intersection all belong on the base.
- If a second provider in the same domain would want code you're writing inside a concrete provider, **hoist before landing**.

### [Adding a new provider](./CONTRIBUTING.md#introducing-a-new-provider)

- Extend the domain's existing base class (`LendProvider`, `SwapProvider`). Never roll a parallel abstraction.
- **One protocol version per provider, one domain per PR.** No bundled V2 + V3, no bundled Lend + Borrow.
- For forks of supported protocols (Spark of Aave, Uniswap V3 forks, Compound forks), introduce a shared intermediate base class. Don't copy-paste a full provider.
- Implement protected `_methods` (`_openPosition`, `_getQuote`, …); public methods stay on the base.
- Declare chain support via `protocolSupportedChainIds()`.
- Avoid pulling in the protocol's full SDK unless it provides material correctness or ergonomics value. Prefer viem + ABI snippets.
- Protocol code lives under `packages/sdk/src/actions/<domain>/providers/<protocol>/`. When a protocol first spans multiple domains (e.g. Aave Lend + Borrow), extract its shared constants/ABIs/registries to a cross-domain home in its own small PR ahead of the second domain. Pick the location when the need arises; no shared cross-domain home exists yet.
- Config shape (`<Protocol><Domain>ProviderConfig`) parallels sibling providers.
- Register in `packages/sdk/src/types/providers.ts` under the right domain record. Wire into `Actions` (`packages/sdk/src/actions.ts`) and the `Wallet` / `SmartWallet` classes (`packages/sdk/src/wallet/core/wallets/`) following the existing construction pattern.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ethereum-optimism/actions](https://github.com/ethereum-optimism/actions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
