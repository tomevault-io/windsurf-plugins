---
trigger: always_on
description: >-
---


# FHEVM Development Skill

> Build confidential smart contracts with Fully Homomorphic Encryption on EVM chains using the Zama Protocol.

## Installation

Copy the `fhevm-skill/` directory to your AI tool's skills folder:
- **Claude Code**: `~/.claude/skills/fhevm/`
- **Cursor**: `.cursor/skills/fhevm/`
- **Windsurf**: `.windsurf/skills/fhevm/`
- **Copilot**: `.github/skills/fhevm/`

## Architecture

FHEVM uses a **coprocessor model** with 4 components:

```
User (browser)                          Coprocessor (FHE engine)
  │ encrypt via Relayer SDK                  │ executes actual FHE math
  ▼                                          ▲
Contract (Host Chain, EVM)  ──handles──►     │
  │ stores handles (bytes32)                 │
  │ symbolic execution only                  │
  ▼                                          │
ACL Contract ◄──── who can read what ──────► │
  │                                          │
  ▼                                          │
Gateway Chain ──── orchestrates ──────► KMS (Key Management Service)
                   decryption            │ threshold decryption
                                         │ multi-party computation
                                         │ no single party has the full key
```

**How it works:**

1. **Host Chain** (Ethereum/Sepolia): Your contracts live here. FHE operations are **symbolic** — when you call `FHE.add(a, b)`, the chain produces a new handle but the real encrypted computation happens asynchronously in the coprocessor.

2. **Coprocessor**: Rust-based engine that executes the actual FHE math offchain. Receives operations from the host chain, processes ciphertexts, returns result handles.

3. **ACL Contract**: On-chain registry tracking who can access each encrypted handle. Every `FHE.allow()`, `FHE.allowThis()`, and `FHE.allowTransient()` writes to this contract.

4. **KMS (Key Management Service)**: Handles decryption via **threshold multi-party computation** — the FHE secret key is split across multiple KMS nodes. No single node can decrypt alone. When `FHE.makePubliclyDecryptable()` is called, the KMS nodes cooperatively produce a decryption proof that can be verified on-chain via `FHE.checkSignatures()`.

5. **Gateway Chain** (chainId 10901 for Sepolia): Orchestrates communication between the host chain and KMS for decryption requests.

6. **Relayer SDK** (`@zama-fhe/relayer-sdk`): Frontend library that handles client-side encryption (ZK proof generation) and coordinates decryption requests with the KMS via the Gateway.

**Key implications:**
- You NEVER see plaintext in contract logic (except at system boundaries like wrap/unwrap)
- All branching on encrypted values uses `FHE.select()`, not `if/else`
- Decryption is **asynchronous** — mark a value as decryptable, then verify the KMS proof separately
- The FHE key is **never held by a single entity** — threshold security via KMS

## Picking the Right SDK Layer

Solidity has one current library (`@fhevm/solidity`); the off-chain stack has two layers, both first-class but for different jobs.

### Solidity (on-chain)

| | OLD (deprecated) | CURRENT (use this) |
|---|---|---|
| **Package** | `fhevm` v0.5-0.6 | `@fhevm/solidity` v0.11+ |
| **Library** | `TFHE` | `FHE` |
| **Import** | `import "fhevm/lib/TFHE.sol"` | `import {FHE, euint64} from "@fhevm/solidity/lib/FHE.sol"` |
| **Input type** | `einput` | `externalEuint64` (typed per width) |
| **Input parse** | `TFHE.asEuint64(einput, proof)` | `FHE.fromExternal(externalEuint64, proof)` |
| **Config** | `SepoliaZamaFHEVMConfig` | `ZamaEthereumConfig` |
| **Decryption** | `Gateway.requestDecryption()` | `FHE.makePubliclyDecryptable()` + `FHE.checkSignatures()` |

> **Warning**: `fhevm-contracts` was **archived in 2025** and used the OLD `TFHE` library. It has been replaced by `@openzeppelin/confidential-contracts` which uses the new `FHE` library. Use the new package for all development.

### Off-chain (tests + frontends)

Two layers cooperate — pick by the job, not by "which is newer". The high-level layer is built on top of the foundational layer; they aren't substitutes.

| Layer | Package | Use it for | Don't use for |
|---|---|---|---|
| **Foundational SDK** | `@zama-fhe/relayer-sdk@0.4.1` (EXACT pin) | Every Hardhat test (the plugin imports it internally), server-side scripts, frontends for non-token contracts (voting / auction / AMM / vault UIs), manual encryption pipelines | Token UIs where the high-level layer is more ergonomic |
| **High-level Token API** | `@zama-fhe/sdk@3.x` + `@zama-fhe/react-sdk@3.x` | ERC-7984 token UIs in the browser (balances, transfers, wraps, operator setup), React apps wanting `useConfidentialBalance` / `useConfidentialTransfer` hooks | Hardhat tests, non-token contracts, server-side encryption |
| **Deprecated** | `fhevmjs` | Never. Migration table below. | — |

**Rule of thumb:** The foundational SDK is mandatory at the test layer (the hardhat-plugin pins it). Add the high-level Token API on top whenever you build a token UI. They're independent and can coexist in one app.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Himess/fhevm-skill](https://github.com/Himess/fhevm-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
