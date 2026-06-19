---
trigger: always_on
description: Access AIsa x402-paid /apis/v2/ endpoints using mainnet USDC and Circle Gateway across 11 EVM chains (Ethereum, Base, Avalanche, Arbitrum, OP, Polygon, Unichain, Sonic, World Chain, Sei, HyperEVM). Use when setting up x402 payments, creating or funding a wallet, depositing into Circle Gateway on a specific chain, picking the right AIsa endpoint for a task, estimating per-call cost, or making paid AIsa API calls without an API key. 104 endpoints across Twitter, Financial, Search, Scholar, Perplex
---


# nanopayment-x402

Pay-per-call API access to 104 AIsa endpoints via the x402 HTTP payment protocol. No API key needed — pays with USDC on any of 11 EVM mainnets via Circle Gateway. (Arc Testnet is in the chain registry for legacy wallets but is no longer accepted by AIsa for paid endpoints.)

## How It Works

```
Agent ──► AIsa API (HTTP 402) ──► Agent signs EIP-712 payment ──► API returns data
                                         │
                               Circle Gateway (batched USDC settlement)
```

1. Agent sends a request to a paid `/apis/v2/` endpoint
2. Server responds with HTTP 402 + a `payment-required` header containing accepted payment networks and amounts
3. Agent signs an EIP-712 `TransferWithAuthorization` for USDC via Circle's GatewayWalletBatched contract
4. Agent re-sends the request with the signed payment in headers
5. Server verifies the signature, settles via Circle Gateway, and returns data

> **Note:** The AIsa proxy uses a custom EIP-712 domain where `verifyingContract` is the Gateway contract (from `extra.verifyingContract` in the 402 response), not the USDC asset address. The standard `@x402/evm` `ExactEvmScheme` does not handle this — the included `GatewayEvmScheme` in `x402_client.mjs` handles it.

## Quick Reference

| Item | Value |
|------|-------|
| API Base | `https://api.aisa.one/apis/v2/` |
| Accepted chains (mainnet) | `ethereum` `base` `avalanche` `arbitrum` `optimism` `polygon` `unichain` `sonic` `worldchain` `sei` `hyperevm` |
| Chain selection | `--chain <key\|id>` flag or `OWS_CHAIN` env (no default — pick explicitly) |
| Gateway Wallet (mainnet) | `0x77777777Dcc4d5A8B6E418Fd04D8997ef11000eE` (same on every EVM mainnet) |
| USDC | 6 decimals on every chain. Per-chain USDC contract addresses in `scripts/chains.mjs` |
| Per-chain RPC override | `OWS_RPC_<KEY>` env (e.g. `OWS_RPC_BASE`) |
| Chain registry | `scripts/chains.mjs` |
| Endpoint catalog | `references/endpoint-catalog.md` |
| API Path Prefix | `/apis/v2/` (x402) vs `/apis/v1/` (API key) |
| List chains | `node scripts/setup.mjs chains` |

## Prerequisites

- **Node.js** (v18+)
- **npm**

```bash
# Install skill dependencies
npm install
```

## Decision Flow

On every invocation, execute this sequence:

### 1. Check Prerequisites

```bash
bash scripts/check-env.sh
```

If `node`, `npm`, or deps are missing:
```bash
npm install
```

### 2. Ensure Wallet Exists

**If mnemonic found** (check in order: `OWS_MNEMONIC` env, `X402_MNEMONIC` env, local `.env`): proceed to step 3.

**If no mnemonic found**, generate a wallet directly using viem (no interactive terminal needed) and save it to `.env` in one step:

```bash
node --input-type=module -e "
import { generateMnemonic, english, mnemonicToAccount } from 'viem/accounts';
import fs from 'fs';
import path from 'path';
const mnemonic = generateMnemonic(english);
const account = mnemonicToAccount(mnemonic);
const envPath = path.resolve('.env');
fs.writeFileSync(envPath, 'OWS_MNEMONIC=' + mnemonic + '\n');
console.log('MNEMONIC_SAVED=true');
console.log('ADDRESS=' + account.address);
"
```

This generates a BIP-39 mnemonic, derives the Ethereum address, and persists the mnemonic to `.env` — fully automated with no user interaction.

**Mainnet funding policy:** AIsa-accepted chains are mainnets. There is no faucet — the user must fund the wallet with real USDC + native gas. Do NOT attempt to automate funding in a browser.

Get the wallet address:
```bash
node scripts/setup.mjs address
```

Display the wallet address prominently and tell the user:

```
Your new wallet address (click to copy):

`0x<WALLET_ADDRESS>`

Fund this wallet with USDC (and a small amount of native gas) on one of the
11 AIsa-accepted EVM mainnets — e.g. transfer USDC from an exchange or another
wallet to this address on Base, Arbitrum, OP, etc. Pick whichever chain has the
cheapest gas + lowest USDC bridge cost for your situation; the Gateway unifies
balances across all of them.

Run `node scripts/setup.mjs chains` to see the full list.
```

Wait for the user to confirm they have funded the wallet, ask which chain they sent to, then verify:

```bash
node scripts/setup.mjs balance --chain <key>
# or to scan every chain in the registry:
node scripts/setup.mjs balance --all
```

If ERC-20 USDC is still `0` on the chain they specified, the transfer may not have landed — ask them to wait for confirmation or recheck the destination chain.

Once funded, continue to step 3 to approve and deposit into the Gateway on that chain.

### 3. Check Balance and Auto-Deposit

First, identify the active chain — `OWS_CHAIN` env, prior conversation, or ask the user. Then:

```bash
node scripts/setup.mjs balance --chain <key>
# Or scan every chain in the registry:
node scripts/setup.mjs balance --all
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AIsa-team/nanopayment-x402](https://github.com/AIsa-team/nanopayment-x402) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
