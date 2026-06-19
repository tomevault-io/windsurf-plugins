---
trigger: always_on
description: Use this skill whenever the user asks any question about crypto assets, blockchain networks, DeFi protocols, token unlocks, fundraising rounds, market data, social sentiment, news, or governance — and live data from the Messari API would improve the answer. This includes questions like \"what's the price of X\", \"who invested in Y\", \"what are the top DeFi protocols by TVL\", \"what's trending in crypto\", \"show me upcoming token unlocks\", \"what's the latest news on Ethereum\", \"compare L1
---


# Messari — Leading Crypto Data Platform

Answer crypto questions using the Messari API: 34,000+ assets, 210+ exchanges, 14 data services
covering market metrics, social sentiment, research, on-chain analytics, fundraising, governance,
and more.

**Base URL for all endpoints:** `https://api.messari.io`

---

## Step 1: Confirm Authentication

Follow this decision tree **before every session**. Do not skip to Step 2 until auth is confirmed.

```
1. Is `payments-mcp` available as a tool?
   ├── YES → Run `payments-mcp:get_wallet_balance`
   │          ├── Success → x402 is ready. Proceed to Step 2.
   │          └── Error   → Run `payments-mcp:show_wallet_app`, then prompt user (see Setup below)
   └── NO  → Is $MESSARI_API_KEY set or has the user provided an API key?
              ├── YES → API key mode. Include `x-messari-api-key: <key>` on all requests. Proceed to Step 2.
              └── NO  → Neither auth is configured. Prompt user (see Setup below).
```

### x402 Setup (Recommended — wallet-based, no API key needed)

If x402 is not yet configured, tell the user:

> **To use Messari, you'll need to set up a payment wallet first. Here's how:**
>
> 1. Run this command to install the payments connector (replace `claude` with your client if different):
>    ```
>    npx @coinbase/payments-mcp --client claude --auto-config
>    ```
>    Supported clients: `claude` | `claude-code` | `codex` | `gemini`
>
> 2. Restart your AI client to load the connector.
>
> 3. Come back and I'll open the wallet app for you to sign in.
>
> Once signed in, deposit some Base USDC to cover API requests (costs are fractions of a cent per call).

After the user has installed and restarted, run `payments-mcp:show_wallet_app` to open the wallet and prompt them to sign in and deposit USDC.

### API Key (Alternative)

If the user has a Messari API key, include this header on every request:
```
x-messari-api-key: <MESSARI_API_KEY>
```
Note: some endpoints are **only** available via API key (marked `api_key only` below).

---

## Step 2: Route to the Right Service

| User is asking about... | Service | Auth |
|---|---|---|
| General crypto question, synthesis, open-ended research | **AI** | api_key, x402 |
| Price, volume, market cap, ROI, ATH, performance comparison | **Metrics** | api_key, x402 |
| Sentiment, mindshare, trending tokens, social buzz | **Signal** | api_key, x402 |
| Headlines, recent events, breaking news | **News** | api_key, x402 |
| Analyst reports, deep dives, sector overviews | **Research** | api_key only |
| Stablecoin supply, flows, chain breakdowns | **Stablecoins** | api_key, x402 |
| Exchange volumes, comparisons | **Exchanges** | api_key only |
| L1/L2 network activity, fees, active addresses | **Networks** | api_key, x402 |
| DeFi protocols, TVL, lending, DEX volume | **Protocols** | api_key only |
| Token unlocks, vesting schedules | **Token Unlocks** | api_key, x402 |
| Fundraising rounds, investors, VC activity, M&A | **Fundraising** | api_key, x402 |
| Governance events, protocol upgrades | **Intel** | api_key only |
| Trending narratives, topic momentum | **Topics** | api_key only |
| Crypto influencers, X/Twitter accounts | **X-Users** | api_key, x402 |

**When in doubt, start with the AI service** — it synthesizes across all sources and handles
open-ended questions well.

### Example query routing
```
"Tell me about x402 and how it works."              → AI       /ai/v2/chat/completions
"What are upcoming token unlocks this month?"       → Unlocks  /token-unlocks/v1/assets
"10 most recent AI/compute fundraising rounds"      → Funding  /funding/v1/rounds
"Latest crypto regulation headlines"                → News     /news/v1/news/feed
"Recent DePIN sector developments"                  → Research /research/v1/reports
"Most active seed investor over last year"          → Funding  /funding/v1/rounds/investors
"Top AAVE events last quarter"                      → Intel    /intel/v1/events
"Solana ecosystem map"                              → Networks /metrics/v2/networks
"Recent a16z crypto investments"                    → Funding  /funding/v1/projects
"Compare BitTensor vs Render native assets"         → Metrics  /metrics/v2/assets/details
```

---

## Step 3: Read detailed endpoint documentation and call the endpoint

After deciding which service to use, read the detailed endpoint documentation from url in the "spec" column below before calling the endpoint. 

Many endpoints require an assetID, for these you should first call the unauthenticated lsit assets endpoint `/metrics/v2/assets?limit=200` with `curl` to get the assetID.

### AI Service
Chat completions trained on 30TB+ of structured crypto data. Handles synthesis, comparisons,
and open-ended research. x402 access does not require pre-purchased credits.

| Endpoint | Method | Spec |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [messari/skills](https://github.com/messari/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
