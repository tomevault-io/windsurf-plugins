---
trigger: always_on
description: This skill is derived from building **Obscura** - a confidential RFQ dark pool on Zama FHEVM, deployed live on Sepolia at [obscuradex.vercel.app](https://obscuradex.vercel.app). 21-day sprint, 47 contract tests passing, full encrypted match-and-settle proven on-chain from a browser.
---

# Zama FHEVM Builder Skill - Confidential dApp Survival Kit

> Agent-usable playbook for shipping a confidential application on Zama FHEVM.
> Stack: Solidity FHEVM ops, OpenZeppelin ERC-7984, Vue 3 + Vite, Wagmi-Vue, viem, `@zama-fhe/relayer-sdk`.

---

## Context

This skill is derived from building **Obscura** - a confidential RFQ dark pool on Zama FHEVM, deployed live on Sepolia at [obscuradex.vercel.app](https://obscuradex.vercel.app). 21-day sprint, 47 contract tests passing, full encrypted match-and-settle proven on-chain from a browser.

Source repo: [github.com/unifyWeb3/obscura-monorepo](https://github.com/unifyWeb3/obscura-monorepo)

Every trap in this document came from a real bug, a real wasted hour, or a real production failure. The format is intentional: **Detection → Action → Why it works**, so an AI agent (or developer) can pattern-match against symptoms and apply the fix without re-deriving the underlying reasoning.

---

## Skill overview

This skill enables an agent or developer to:

1. Bootstrap a Vue 3 + Wagmi + relayer-sdk frontend that talks to FHEVM contracts on Sepolia
2. Encrypt user inputs client-side, submit them on-chain with ZK proofs, decrypt results via threshold KMS
3. Avoid the 23 documented integration traps that consume days of debugging time
4. Deploy to Vercel with the cross-origin isolation headers the relayer-sdk requires
5. Make confident architectural decisions (when to self-relay vs operator-relay, when to use plaintext vs ciphertext, how to expose state to the user)

---

## When to use this skill

Apply this skill when:

- Building any dApp where **user inputs need to stay private** during on-chain processing (RFQ, dark pools, sealed-bid auctions, private voting, confidential payments)
- The frontend stack is Vue 3 (with Wagmi-Vue) - most React-first FHEVM tutorials do NOT translate cleanly
- You're integrating `@zama-fhe/relayer-sdk` for browser-side encryption and threshold KMS public decryption
- You're deploying to Vercel or any static host where you control HTTP headers
- The contract uses ERC-7984 confidential token transfers as part of settlement

Do NOT use this skill for:

- Pure smart contract work without a frontend (skip the Wagmi/Vite traps)
- React-based dApps (the Vue-specific patterns will mislead you; the relayer-sdk traps still apply)
- FHEVM mainnet deployments (this skill targets Sepolia testnet - production has additional KMS configuration concerns)

---

## Core stack

| Layer               | Tool                                | Version         | Why this choice                                                    |
| ------------------- | ----------------------------------- | --------------- | ------------------------------------------------------------------ |
| Smart contracts     | Solidity + FHEVM Solidity SDK       | `0.11.x`        | Required for FHE ops                                               |
| Confidential tokens | OpenZeppelin Confidential Contracts | `0.4.x`         | ERC-7984 reference impl                                            |
| Contract testing    | Hardhat + FHEVM mock                | latest          | Mock skips KMS, runs in CI                                         |
| Frontend framework  | Vue 3.5                             | latest          | TypeScript-first, smaller bundles than React for crypto-heavy apps |
| Build tool          | Vite 8                              | latest          | Required for relayer-sdk WASM workers                              |
| Wallet & contracts  | `@wagmi/vue` + `viem`               | `0.5.x` / `2.x` | Vue-native bindings around viem                                    |
| Encryption + KMS    | `@zama-fhe/relayer-sdk`             | `0.4.x`         | Web entry - browser-side encryption + ZK proofs + threshold KMS    |
| Styling             | Tailwind CSS v4                     | latest          | Custom design tokens via `@theme`                                  |
| Deployment          | Vercel                              | latest          | Native Vite preset, configurable headers                           |

---

## Mental models

### How FHEVM actually behaves

1. **Plaintext never leaves the browser.** The relayer-sdk encrypts user inputs locally using the FHEVM public key + generates a ZK proof of the encryption. The contract receives ciphertext handles + the proof.

2. **The contract operates on ciphertext.** `euint64`, `euint32`, `ebool` are encrypted types. Comparisons (`FHE.gt`, `FHE.lt`, `FHE.eq`) return encrypted booleans. Conditional logic uses `FHE.select(cond, ifTrue, ifFalse)` - both branches always compute, only the result is selected obliviously.

3. **No actor sees the cleartexts during computation.** Not the operator, not the contract author, not any node operator. The encrypted handles are processed by the FHEVM coprocessor.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unifyWeb3/obscura](https://github.com/unifyWeb3/obscura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
