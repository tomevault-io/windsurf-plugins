---
trigger: always_on
description: Monorepo (pnpm workspaces) for Babylon's Bitcoin vault frontend. Users lock BTC on Bitcoin, receive vaultBTC on Ethereum for DeFi collateral. The frontend manages the full depositor lifecycle: vault provider selection, deposit, presigning payout transactions, broadcasting, redemption.
---

# CLAUDE.md — babylon-toolkit

## Project Overview

Monorepo (pnpm workspaces) for Babylon's Bitcoin vault frontend. Users lock BTC on Bitcoin, receive vaultBTC on Ethereum for DeFi collateral. The frontend manages the full depositor lifecycle: vault provider selection, deposit, presigning payout transactions, broadcasting, redemption.

### Key Packages
- `services/vault` — Main vault dApp (Next.js)
- `packages/babylon-tbv-rust-wasm` — Rust→WASM for transaction construction, fee calculation
- `packages/wallet-connector` — Multi-chain wallet abstraction (BTC + ETH)
- `packages/core-ui` — Shared UI component library
- `packages/ts-sdk` — TypeScript SDK for protocol interaction

### Build Prerequisites
- Node 24 via nvm (`nvm use 24`), pnpm via Corepack
- Must rebuild `core-ui` and `ts-sdk` before vault build (stale `dist/` is a common issue)

## Build & Test Commands

```bash
nvm use 24                            # Switch to Node 24 (required)
pnpm install                          # Install all dependencies
pnpm run build                        # Build all packages
pnpm run lint                         # Lint all packages
pnpm run test                         # Run all tests (vitest)
pnpm --filter vault run dev           # Dev server for vault service
```

Run `pnpm run lint` and `pnpm run test` in the affected service before considering work done.

---

## CRITICAL PATHS — HUMAN REVIEW REQUIRED

These paths handle irreversible value movement. An AI-generated mistake here is silent: code compiles, tests pass, wrong BTC amount ships. **Any change touching these files requires two reviewers, and the author must be able to explain every changed line without an AI assistant open.**

### 1. WASM boundary (value computation)
- File: `packages/babylon-tbv-rust-wasm/src/index.ts`
- The Rust/WASM layer computes `htlcValue = peginAmount + depositorClaimValue + minPeginFee` internally. JS receives outputs with no runtime validation.
- **Rule:** Every WASM output consumed by JS must be asserted against expected bounds before use. If a WASM-returned value feeds a signed transaction, cross-check it against an independently computed expected value.

### 2. Fee calculation consistency
- Files:
  - `packages/babylon-ts-sdk/src/tbv/core/utils/utxo/selectUtxos.ts` — UTXO selection with iterative fee recalculation
  - `services/vault/src/utils/fee/peginFee.ts` — dApp-side estimate with safety margin
- Both systems must agree before broadcast. A mismatch underfunds the transaction.
- **Rule:** When changing either, re-verify the other produces the same fee for a representative fixture. Cross-check assertions belong at the broadcast site, not only at the estimator.

### 3. Presigning depositor-graph transactions (Payout + NoPayout)
- Files:
  - `packages/babylon-ts-sdk/src/tbv/core/primitives/psbt/payout.ts`
  - `packages/babylon-ts-sdk/src/tbv/core/services/deposit/signDepositorGraph.ts` — orchestrator that derives `LocalChallengers`, asserts the VP-returned `challenger_presign_data` set equals `local ∪ universal`, and decides which per-challenger NoPayout PSBTs get pre-signed
  - `services/vault/src/hooks/deposit/depositFlowSteps/payoutSigning.ts`
- The depositor pre-signs payout (and per-challenger NoPayout) transactions built by the Vault Provider — values and challenger sets come from an external party with no independent verification. Asymmetric failure: undersigning leaves recovery material missing for an active challenger; oversigning hands signatures to a key the protocol doesn't recognize.
- **Rule:** Before the signature call, re-derive the expected payout amount from on-chain or WASM-computed sources and assert equality. For the challenger set, derive `LocalChallengers` from on-chain VK list (matching the Rust reference in `btc-vault crates/vault/src/tx_graph/graph.rs`) and assert the VP-returned set equals `local ∪ universal` exactly — no missing entries, no extras. Never sign a value or accept a challenger key handed to us verbatim.

### 4. Vault-secret derivation (frozen on-chain-binding API)
- Files (all marked `@stability frozen` in JSDoc):
  - `packages/babylon-ts-sdk/src/tbv/core/vault-secrets/context.ts` — `buildVaultContext`, `buildFundingOutpointsCommitment`
  - `packages/babylon-ts-sdk/src/tbv/core/vault-secrets/deriveVaultRoot.ts` — `deriveVaultRoot`, `VAULT_APP_NAME`
  - `packages/babylon-ts-sdk/src/tbv/core/vault-secrets/index.ts` — re-exports `expandAuthAnchor`, `expandHashlockSecret`, `expandWotsSeed` from the WASM package
  - `packages/babylon-tbv-rust-wasm/src/index.ts` — browser-side async wrappers for the three expanders
  - `packages/babylon-tbv-rust-wasm/src/index-node.ts` — node-side async wrappers for the three expanders
  - `packages/babylon-tbv-rust-wasm/scripts/build-wasm.js` — `BTC_VAULT_COMMIT` pin (the Rust crate at this commit is the byte-level source of truth for the HKDF `info` encoding, labels, and i2osp prefixes)
  - `packages/babylon-ts-sdk/src/tbv/core/wots/blockDerivation.ts` — `deriveWotsBlocksFromSeed`, `computeWotsBlockPublicKeysHash`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [babylonlabs-io/babylon-toolkit](https://github.com/babylonlabs-io/babylon-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
