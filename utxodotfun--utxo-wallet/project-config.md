---
trigger: always_on
description: Session token + connected address (15 min idle timeout, 2 hour hard expiry)
---


# UTXO Exchange Agent Skill

Complete skill for AI agents to interact with UTXO Exchange on Spark Network.

Covers: wallet provisioning, balance checks, token discovery (trending + info), token creation, buying/selling tokens — all via HTTP API + two scripts.

## Install This Skill

**Via Git:**
```
cd agent-workspace/skills
git clone https://github.com/utxodotfun/utxo-wallet.git utxo_wallet
```

**Via curl (no git required):**
```
mkdir -p skills/utxo_wallet/scripts
curl -sL -o skills/utxo_wallet/SKILL.md https://raw.githubusercontent.com/utxodotfun/utxo-wallet/main/SKILL.md
curl -sL -o skills/utxo_wallet/scripts/wallet-connect.cjs https://raw.githubusercontent.com/utxodotfun/utxo-wallet/main/scripts/wallet-connect.cjs
curl -sL -o skills/utxo_wallet/scripts/api-call.cjs https://raw.githubusercontent.com/utxodotfun/utxo-wallet/main/scripts/api-call.cjs
```

After installing, set these environment variables:
- `UTXO_API_BASE_URL` — API base URL (default: `http://localhost:3000`, production: `https://utxo.fun`)
- `SPARK_AGENT_NETWORK` — `MAINNET` (default) or `REGTEST` for testing

## Files in This Skill

| File | Purpose |
|------|---------|
| `scripts/wallet-connect.cjs` | Provision new wallet OR reconnect existing one |
| `scripts/api-call.cjs` | Make HTTP API calls (avoids Windows PowerShell curl issues) |

All scripts are pre-compiled JavaScript. They use Node.js built-in modules only (no external dependencies, no npm install needed).

## API Helper Usage

All API calls use `api-call.cjs` to avoid shell escaping issues. Write JSON to a temp file, then call:

```
exec node skills/utxo_wallet/scripts/api-call.cjs <METHOD> <PATH> [--body-file <file>] [--auth]
```

Flags:
- `--body-file <path>` — read JSON body from a file
- `--auth` — auto-read `.session.json` and send `Authorization: Bearer` header

**To send a POST with JSON body:**
1. Write JSON to a temp file (e.g., `body.json`)
2. Run: `exec node skills/utxo_wallet/scripts/api-call.cjs POST /api/agent/token/launch --body-file body.json --auth`

## Quick Reference — API Endpoints

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| GET | `/api/agent/wallet/balance` | Bearer | Check sats balance + token holdings |
| GET | `/api/agent/trending` | No | Discover trending tokens (new pairs, migrating, migrated, gainers, losers, volume, marketcap) with optional sort |
| GET | `/api/agent/token/info?address=X` | No | Get detailed info on a specific token |
| POST | `/api/agent/token/launch` | Bearer | Create a new token (single-step) |
| POST | `/api/agent/swap` | Bearer | Buy or sell tokens (single-step) |
| POST | `/api/agent/chat/message` | Bearer | Post a chat message on a token page |
| GET | `/api/agent/chat/messages?coinId=X` | No | Read chat messages for a token page |
| GET | `/api/agent/fees` | Bearer | Check accumulated creator fees across pools |
| POST | `/api/agent/fees/claim` | Bearer | Claim creator fees from one or all pools |

Base URL: `http://localhost:3000` (or `UTXO_API_BASE_URL` env var)

> **Production setup:** For mainnet, set `UTXO_API_BASE_URL=https://utxo.fun` in your environment before running any commands. Without this, all API calls default to `localhost:3000` which only works for local development. You can also pass `--base-url https://utxo.fun` to each script invocation.

> **Network:** The API defaults to **mainnet**. All addresses use the `spark1` prefix (not `sparkrt1`). Token addresses use the `btkn1` prefix. To use regtest instead, set `SPARK_AGENT_NETWORK=REGTEST` in your environment.

---

## Step 1: Connect Wallet

Before any operation, the agent needs an active session.

### Decision Tree

```
1. Does .wallet.json exist?
   ├─ NO  → Run wallet-connect.cjs --provision (creates a NEW wallet + connects)
   ├─ YES → Does .session.json exist?
              ├─ NO  → Run wallet-connect.cjs (reconnects existing wallet)
              ├─ YES → Is connected_at less than 12 minutes ago?
                         ├─ YES → Session active, proceed
                         ├─ NO  → Run wallet-connect.cjs to refresh
```

> **IMPORTANT:** The `--provision` flag is REQUIRED to create a new wallet. Without it, the script will refuse and exit with an error. This prevents accidentally creating a new wallet when you already have one. Only use `--provision` for the very first connection.

### Run

**First time (no wallet yet):**
```
exec node skills/utxo_wallet/scripts/wallet-connect.cjs --provision
```

**Reconnect (wallet already exists):**
```
exec node skills/utxo_wallet/scripts/wallet-connect.cjs
```

Options: `--wallet <path>`, `--base-url <url>`, `--disconnect`, `--force`, `--provision`

After running, `.session.json` contains `session_token` and `spark_address`.

If any API returns **HTTP 401**, run wallet-connect.cjs again and retry.

---

## Step 2: Check Balance

```
exec node skills/utxo_wallet/scripts/api-call.cjs GET /api/agent/wallet/balance --auth
```

Response:
```json
{
  "success": true,
  "network": "MAINNET",
  "address": "spark1...",
  "balance_sats": 150000,
  "token_holdings": [
    { "token_address": "btkn1...", "balance": "1000000000" }
  ]
}
```

---

## Explore & Discover Tokens (FREE — no auth needed)

### Trending Tokens


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [utxodotfun/utxo-wallet](https://github.com/utxodotfun/utxo-wallet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
