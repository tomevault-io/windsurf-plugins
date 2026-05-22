---
trigger: always_on
description: Guidance for AI coding agents working in this repository. This is the canonical
---

# AGENTS.md

Guidance for AI coding agents working in this repository. This is the canonical
agent-facing file; `CLAUDE.md` points here.

## What this crate is

A Rust SDK for Circle's Cross-Chain Transfer Protocol (CCTP), bridging USDC
across 26+ EVM chains. Supports both CCTP v1 (legacy, 7 chains) and v2 (current,
fast transfers with sub-30s settlement).

## When to use what

| Task | Reach for | Notes |
|---|---|---|
| Bridge USDC on a modern chain | `CctpV2Bridge` | The default. Permissionless mint — see relayer-race note below. |
| Bridge USDC on a v1-only legacy chain | `Cctp` | You extract the message from chain yourself. |
| Submit a burn and let any relayer complete it | `CctpV2Bridge::wait_for_receive` | Cheapest happy path. |
| Submit a burn and try to self-relay | `CctpV2Bridge::mint_if_needed` | Returns `MintResult::AlreadyRelayed` if a relayer beat you. **Do not use raw `mint`** unless you've checked. |
| Check if a transfer already completed | `CctpV2Bridge::is_message_received` | Returns `bool`. |
| Inspect a v2 message as JSON | `ParsedV2MessageSummary::parse` | Returns `ParseMessageError`, not `CctpError`. |
| Drive contracts directly (custom hooks, batch flows) | `TokenMessengerV2Contract` / `MessageTransmitterV2Contract` | Bridge types wrap these. |
| Look up chain config without a provider | `CctpV1` / `CctpV2` traits on `NamedChain` | Pure functions: domain id, addresses, confirmation times. |
| Tune attestation polling | `PollingConfig::fast_transfer()` for v2 fast, `PollingConfig::default()` otherwise | Customize via `with_max_attempts`, `with_poll_interval_secs`. |

## Footguns

- **V2 on-chain messages have a zeroed nonce.** The `MessageSent` event you read
  from the burn receipt is *not* the canonical message. Always use the message
  returned by `CctpV2Bridge::get_attestation`, which fetches the canonical
  version from Circle's Iris API.
- **V2 is permissionless.** Third-party relayers (Synapse, LI.FI, others) watch
  for burns and may complete the mint before you do. Prefer `mint_if_needed`
  over `mint`; the former returns `MintResult::AlreadyRelayed` instead of
  surfacing a confusing on-chain revert.
- **`recipient` is `bytes32`, not `address`.** It's the 20-byte address
  left-padded to 32 bytes. The builder accepts an `Address` and handles padding;
  if you go around the builder, do the padding yourself.
- **Mainnet vs testnet hit different Iris hosts** (`iris-api.circle.com` vs
  `iris-api-sandbox.circle.com`). Selection is automatic from the chain — but
  if you stub the API in tests, stub both.

## Public API map

All exports live in `src/lib.rs` under `pub use`. Quick map for navigation:

| Type / function | Purpose | Source |
|---|---|---|
| `Cctp`, `CctpBridge` | V1 bridge struct + trait | `src/bridge/cctp.rs`, `src/bridge/bridge_trait.rs` |
| `CctpV2Bridge` (re-export of `CctpV2`) | V2 bridge with fast-transfer support | `src/bridge/v2.rs` |
| `MintResult` | `Minted(TxHash)` / `AlreadyRelayed` | `src/bridge/v2.rs` |
| `PollingConfig` | Attestation polling tuning | `src/bridge/config.rs` |
| `TokenState`, `batch_token_state` | ERC-20 allowance/balance helpers | `src/bridge/` |
| `CctpV1`, `CctpV2` traits | Chain config on `NamedChain` | `src/chain/config.rs`, `src/chain/v2.rs` |
| `CCTP_V2_*_MAINNET/TESTNET` | Unified v2 contract addresses | `src/chain/addresses.rs` |
| `TokenMessengerContract`, `MessageTransmitterContract` | V1 contract wrappers | `src/contracts/` |
| `TokenMessengerV2Contract`, `MessageTransmitterV2Contract` | V2 contract wrappers | `src/contracts/v2/` |
| `Erc20Contract` | Minimal ERC-20 wrapper | `src/contracts/erc20.rs` |
| `AttestationResponse`, `AttestationStatus`, `V2AttestationResponse` | Iris API response types | `src/protocol/attestation.rs` |
| `ParsedV2Message`, `ParsedV2MessageSummary`, `BurnMessageV2`, `MessageHeader`, `V2Message` | Canonical v2 message parsing | `src/protocol/message.rs` |
| `DomainId`, `FinalityThreshold` | Protocol constants | `src/protocol/` |
| `CctpError`, `Result`, `ParseMessageError` | Error types | `src/error.rs`, `src/protocol/message.rs` |
| `ProviderConfig*`, `estimate_gas_with_buffer`, `calculate_gas_price_with_buffer` | Provider tuning helpers | `src/provider.rs` |
| `spans` module | OpenTelemetry instrumentation hooks | `src/spans.rs` |

## Project structure

`src/lib.rs` defines the public SDK surface. Bridge logic lives in `src/bridge/`
(`cctp.rs` for v1, `v2.rs` for v2, `config.rs` for polling, `bridge_trait.rs`
for the shared trait). Chain metadata is in `src/chain/`, protocol parsing and
types in `src/protocol/`, contract wrappers in `src/contracts/`, and shared
errors in `src/error.rs`. Runnable samples go in `examples/`, contract ABI
JSON in `abis/`, and CI/release automation in `.github/workflows/`.

## Build, test, and development commands

- `cargo build --all-targets --all-features` — builds library, tests, examples.
- `cargo test --all-features --verbose` — runs the full test suite.
- `cargo test --doc --all-features --verbose` — checks doctests.
- `cargo clippy --all-targets --all-features -- -D warnings` — matches the CI lint gate.
- `cargo fmt --all -- --check` — verifies formatting; `cargo fmt` applies fixes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [semiotic-ai/cctp-rs](https://github.com/semiotic-ai/cctp-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
