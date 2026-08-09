---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A provably-fair **gacha** (loot-box / pack-pull) game on Solana, modeled on how
platforms like Collector Crypt and Phygitals actually work — and wired into
Collector Crypt's real [`cc-vrf`](https://vrf.collectorcrypt.com) registry
program by CPI. An admin configures a pool of fixed-weight reward tiers and a
fixed entry fee, and registers an off-chain VRF operator. A buyer pays the fee
to open a pull, committing buyer-supplied entropy into the VRF input. The
operator reveals the ECVRF output (`beta`); the program anchors the proof in the
cc-vrf registry, expands `beta` into a weighted tier, and the prize — a
Token-2022 NFT whose metadata carries a `rarity` field — is minted to the buyer.

## Randomness model (important)

RFC 9381 `ECVRF-EDWARDS25519-SHA512-TAI`, following Collector Crypt's cc-vrf.
**Solana cannot verify an ECVRF proof on-chain** (no precompile), so the trust
model is _detection, not prevention_: on-chain the program accepts the
registered operator's signed `beta`; off-chain anyone can prove cheating. The
design closes every gap that detection alone leaves open:

1. **Fixed ≠ unpredictable.** `alpha = SHA-256(pull_address || client_seed)`
   where `client_seed` is 32 random bytes chosen by the buyer at commit. An
   alpha that is merely _fixed_ (say, the pull address alone) is worthless
   against the operator: `beta = VRF(operator_key, alpha)` is deterministic, so
   a predictable alpha lets the operator precompute every outcome before anyone
   buys. Buyer entropy is what makes the outcome unknowable at commit time.
2. **Fixed weights ⇒ order-independence.** Tier odds never change after init,
   so a pull's outcome depends only on its `beta` — not on supply counters or
   the order in which the operator settles. (Selection against a mutable
   remaining-supply table would let an operator who knows every pending `beta`
   route rare tiers to favored wallets purely by choosing settle order, and
   per-pull proof verification would never catch it.)
3. **One reveal per pull, enforced by cc-vrf.** `settle_pull` CPIs cc-vrf's
   `commit_proof_with_beta`, whose Light Protocol compressed account derives
   from `(authority, memo_hash = SHA-256(pull_address))` — a second commit for
   the same pull fails at the Light system program. The commit also proves (via
   validity proof against the registry) that the operator's authority record is
   **frozen**, unrevoked, and keyed by exactly `pool.operator`.
4. **Liveness has an escape hatch.** An operator can withhold a reveal (e.g.
   after privately computing an unfavorable `beta`), but `refund_pull` returns
   the buyer's entry fee and rent after `settle_deadline_slots`, and
   `withdraw_fees` can never touch pending buyers' escrow (the vault reserves
   `pending_pulls × entry_fee`). Withholding delays; it never steals.
5. **Verification story.** From the emitted events anyone can: recompute
   `alpha` from `(pull, client_seed)`, verify the proof with
   `@collectorcrypt/ecvrf`, reproduce the tier with `selectTier`, and check the
   registry commit. The operator's 32-byte Ed25519 seed is **both** its Solana
   signing key and its ECVRF key, so `pool.operator` equals the ECVRF public key.

Comparison: oracle VRFs (Switchboard On-Demand, MagicBlock VRF, ORAO) verify the
randomness proof **on-chain** at the cost of oracle fees, extra latency, and an
oracle-network liveness dependency. The cc-vrf pattern is cheaper and
self-operated, but trust is detection-based. Trust notes: cc-vrf's upgrade
authority was **not** renounced as of 2026-07-29 (contradicting its docs), and
its repo declares MIT but commits no LICENSE file.

## Required Versions

- **Rust**: See `rust-toolchain.toml`
- **Node.js**: See `.nvmrc`
- **pnpm**: See `package.json` `packageManager` field
- **Light CLI**: pinned in `justfile` (`zk_cli_version`), installed by `just setup`

## Build Commands

```bash
just build              # program .so → IDL → TS client → dist
just generate-idl       # Generate IDL via Codama (cargo build with build.rs)
just generate-clients   # Generate TypeScript + Rust clients from IDL
just build-program      # Build .so binary only (cargo build-sbf)
just test               # unit + integration + light + client tests
just unit-test          # Rust host unit tests (selection + alpha derivation)
just integration-test   # LiteSVM integration tests (builds the .so first)
just light-test         # Light-stack tests: real settle -> cc-vrf CPI w/ proofs
just dump-cc-vrf        # Fetch the mainnet cc-vrf binary for light-test
just client-test        # TypeScript client tests (parity + ECVRF + forged-reveal)
just burst-test 200     # Devnet: buy+settle 200 pulls, score the reveals + distribution
just burst-report       # Re-score every pull the burst pool has recorded (no txs)
just demo               # Off-chain operator/verifier demo (no RPC)
just fmt                # cargo fmt + prettier
just check              # fmt-check + lint-check
```

## Architecture

Solana program using **Pinocchio** (lightweight `no_std` framework) with **Codama**
for IDL-driven client generation.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [solana-developers/program-examples](https://github.com/solana-developers/program-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
