---
trigger: always_on
description: This document describes how AI agents should interact with the Algorand blockchain through the Algorand MCP server. It covers tool usage patterns, transaction workflows, safety rules, and common pitfalls.
---

# Algorand MCP — Agent Guide

This document describes how AI agents should interact with the Algorand blockchain through the Algorand MCP server. It covers tool usage patterns, transaction workflows, safety rules, and common pitfalls.

## Overview

Algorand MCP is a **local** MCP server that runs on the user's machine. It provides 121 tools across 14 categories for full Algorand blockchain access. Private keys are stored in the **OS keychain** — they never appear in tool responses, logs, or MCP messages.

**Key difference from remote MCP servers**: This server runs locally, signing happens on the user's machine using OS keychain-stored keys, and the agent provides the `network` parameter (`mainnet`, `testnet`, or `localnet`) on each tool call.

## Calling MCP Tools

MCP tools are **deferred** — you MUST use `ToolSearch` to load them before calling:

```
ToolSearch("+algorand wallet")                                    # Search by keyword — loads matching tools
ToolSearch("select:mcp__algorand-mcp__wallet_get_info")           # Load a specific tool by full name
```

Once loaded, call them normally:
```
mcp__algorand-mcp__wallet_get_info { "network": "testnet" }
```

Full tool name pattern: `mcp__algorand-mcp__<tool_name>`. If you get "tool not found", use `ToolSearch("+algorand <keyword>")` to load it first.

## Session Start

At the beginning of every Algorand session:

1. **Load tools** — Call `ToolSearch("+algorand wallet")` to load wallet tools
2. **Check wallet** — Call `wallet_get_info` with the target network to verify a wallet account exists and is active.
3. **If no accounts** — Guide the user to create one with `wallet_add_account` (sets nickname and spending limits).
4. **If account needs funding** — Generate an ARC-26 QR code with `generate_algorand_qrcode` (returns `{ qr, uri, link, expires_in }` — display the `qr` text and shareable `link`) or direct the user to the testnet faucet: https://lora.algokit.io/testnet/fund
5. **If account needs USDC funding** — Direct to testnet USDC faucet: https://faucet.circle.com/

## Network Selection

Every tool that interacts with the blockchain accepts a `network` parameter:

| Value | Description |
|-------|-------------|
| `mainnet` | Algorand mainnet (default if omitted) — **real value, exercise caution** |
| `testnet` | Algorand testnet — safe for development and testing |
| `localnet` | Local development network (requires `ALGORAND_LOCALNET_URL` env var) |

Always confirm with the user which network to use before transactions. Default to `testnet` during development.

## Amounts and Decimals

| Asset | Unit | 1 Whole Token = |
|-------|------|-----------------|
| ALGO | microAlgos | 1,000,000 |
| USDC (ASA 31566704) | micro-units | 1,000,000 (6 decimals) |
| Custom ASAs | base units | Depends on `decimals` field |

Always check the asset's `decimals` field with `api_algod_get_asset_by_id` before computing amounts.

## Common Mainnet Assets

| Asset | ASA ID | Decimals |
|-------|--------|----------|
| ALGO | native | 6 |
| USDC | 31566704 | 6 |
| USDT | 312769 | 6 |
| goETH | 386192725 | 8 |
| goBTC | 386195940 | 8 |

> Always verify asset IDs on-chain — scam tokens use similar names.

## Pre-Transaction Checklist

Before ANY transaction:

1. **MBR (Minimum Balance Requirement)** — Account needs 0.1 ALGO base + 0.1 ALGO per asset opt-in + 0.1 ALGO per app opt-in.
2. **Asset opt-in** — Verify with `api_algod_get_account_asset_info` before ASA transfers. If not opted in, opt-in first.
3. **Fees** — Every transaction costs 0.001 ALGO (1,000 microAlgos) minimum.
4. **Balance check** — Fetch current balance with `wallet_get_info` or `api_algod_get_account_info` before signing.
5. **Spending limits** — The wallet enforces per-transaction (`allowance`) and daily (`dailyAllowance`) spending limits set at account creation. Setting either to `0` means **unlimited**.
6. **Order** — Fund the account with ALGO first, then do asset transactions.

## Transaction Types

| Type | Code | Tools |
|------|------|-------|
| ALGO payment | `pay` | `make_payment_txn` |
| Asset transfer / opt-in | `axfer` | `make_asset_transfer_txn` |
| Asset create / config / destroy | `acfg` | `make_asset_create_txn`, `make_asset_config_txn`, `make_asset_destroy_txn` |
| Asset freeze | `afrz` | `make_asset_freeze_txn` |
| Application (smart contract) | `appl` | `make_app_create_txn`, `make_app_call_txn`, `make_app_update_txn`, `make_app_delete_txn`, `make_app_optin_txn`, `make_app_closeout_txn`, `make_app_clear_txn` |
| Key registration | `keyreg` | `make_keyreg_txn` |

## Wallet Transaction Workflow (Recommended)

Use the wallet tools for signing — they enforce spending limits and keep keys in the OS keychain.

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `wallet_get_info` | Verify active account, check balance |
| 2 | Query tools | Get blockchain data (account info, asset info, etc.) |
| 3 | `make_*_txn` | Build the transaction |
| 4 | `wallet_sign_transaction` | Sign with active wallet account (enforces limits) |
| 5 | `send_raw_transaction` | Submit signed transaction to network |
| 6 | Query tools | Verify result on-chain |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GoPlausible/algorand-mcp](https://github.com/GoPlausible/algorand-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
