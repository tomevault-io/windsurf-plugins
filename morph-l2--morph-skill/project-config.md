---
trigger: always_on
description: AI Agent skill for Morph L2 — wallet, explorer, DEX swap, cross-chain bridge with order management, EIP-8004 agent identity & reputation, alt-fee gas payment, EIP-7702 delegation, and x402 payment protocol
---


# Morph Skill — AI Agent Reference

> CLI toolkit for AI agents to interact with the **Morph Mainnet** (Chain ID: 2818).
> All commands output JSON. All amounts use human-readable units (e.g. `0.1` ETH, not wei).

## Quick Start

```bash
# Install dependencies
pip install requests eth_account eth_abi eth_utils

# Run any command
python3 scripts/morph_api.py <command> [options]
```

No API keys required for queries. Bridge order management requires JWT authentication via `bridge-login`.

---

## Must Read First

Before executing any Morph workflow, decide whether the user is asking for:

- a **Morph protocol/business** task
- a **wallet/Social Login Wallet** task
- or a combined flow that needs both Morph and BGW skills

This repo is the **Morph protocol and business layer**. BGW should be treated as the **wallet product and signing layer**.

- Morph owns wallet RPC operations, explorer queries, DEX quotes, bridge quotes/orders, altfee, and EIP-8004 identity/reputation logic.
- BGW owns Social Login Wallet (TEE signing), swap execution across chains, token discovery, market data, and security audits.
- This repo does **not** call BGW scripts, embed BGW tooling, or manage BGW sessions at runtime.

If the user may need Social Login Wallet behavior, load both the Morph skill pack and the BGW skill pack. BGW scripts live in a separate repo. To locate BGW: check `BGW_DIR` env var → look for `bitget-wallet-skill/` as a sibling directory → if not found, auto-clone from `https://github.com/bitget-wallet-ai-lab/bitget-wallet-skill.git` to the sibling directory. See [docs/social-wallet-integration.md](docs/social-wallet-integration.md) for the full setup flow.

See [docs/social-wallet-integration.md](docs/social-wallet-integration.md) before handling combined Morph + BGW workflows.

### Routing Table

| User Need | Use |
|-----------|-----|
| Local private-key wallet on Morph | Morph skills |
| Explorer, swap, bridge, altfee, identity, reputation on Morph (with local key) | Morph skills |
| EIP-7702 delegation, batch calls (with local key) | Morph skills |
| x402 payment (pay or receive USDC, with local key) | Morph skills |
| x402 discover / verify / settle / server (no signing needed) | Morph skills |
| Social Login Wallet, TEE signing, market data, token discovery | BGW skills |
| Swap/bridge execution with Social Login Wallet (including on Morph) | **BGW skills** — BGW supports Morph chain natively with TEE signing |
| Social Login Wallet + Morph protocol reads | BGW for address, then Morph for reads |
| x402 pay with Social Login Wallet | Agent orchestration: Morph `x402-discover` → BGW signs EIP-3009 → Agent replays with `PAYMENT-SIGNATURE` header |
| EIP-7702 batch with Social Login Wallet | Agent orchestration: Morph computes hashes → BGW signs via TEE → Agent assembles and broadcasts |

Current execution note:

- Morph write commands require `--private-key` for local signing.
- Social Login Wallet users do not have a local private key (keys live in Bitget's TEE). For writes on Morph with a Social Login Wallet, use BGW's swap flow — see [docs/social-wallet-integration.md](docs/social-wallet-integration.md).
- BGW routing in this phase is a documentation/orchestration model, not a new runtime execution path inside `morph_api.py`.

### Single-Pass Routing Model

Choose exactly one mode at the start of the task and stay in it unless the user changes intent:

1. `morph-local-execution`
   Use Morph directly. The user has provided a private key or explicitly wants local-key self-custody.
2. `bgw-wallet-mode`
   Use BGW directly. The user wants Social Login Wallet, TEE signing, swap execution via BGW, or market data queries.
3. `bgw-address-then-morph-read`
   Use BGW first only to resolve the wallet/address context, then use Morph read commands.
4. `bgw-plus-morph-planning`
   Use BGW for wallet context and Morph for protocol reasoning, but do not imply that Morph already has a BGW-native write execution path.

Do not bounce between BGW and Morph more than once for the same task. Route once, hand off the minimum required context, and continue in the selected mode.

### Fast Routing Rules

- If the user already supplied a private key and wants a Morph action executed now, stay in Morph.
- If the user asks for Social Login Wallet or TEE signing, route to BGW.
- If the user wants to swap/bridge with a Social Login Wallet (even on Morph chain), use BGW's swap flow — it supports Morph natively with TEE signing.
- If the user has a BGW wallet but only needs Morph reads, obtain the address from BGW first and then use Morph commands normally.
- If the user asks for BGW-backed execution inside this CLI, explain that Morph CLI requires `--private-key`; for Social Login Wallet execution, use BGW's swap/sign flows instead.

### Handoff Rule

When handing off from BGW to Morph, only carry forward the minimum context needed:

- wallet address
- network/chain intent
- whether the user wants reads, planning, or immediate execution

Do not restate the entire BGW workflow inside each Morph sub-skill.

---

## Data Sources

| Source | Base URL | Auth |
|--------|----------|------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [morph-l2/morph-skill](https://github.com/morph-l2/morph-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
