---
trigger: always_on
description: API base URL. Defaults to https://api.coinversa.ai.
---


# Coinversa Pulse

Coinversa Pulse is a **read-only crypto intelligence MCP skill** for AI agents.

It lets MCP-compatible clients query Hyperliquid market data, trader behavior, position lifecycles, execution quality, cohort analytics, liquidation data, open interest, builder dex markets, HIP-4 outcome contracts, cross-market asset exposure, and wallet-level trading history.

This skill is designed for **market research and analytics only**.

It does **not** place trades, sign transactions, manage wallets, move funds, approve agents, custody assets, or request private keys.

For current wallet and trade coverage numbers, call `pulse_global_stats`.

Coinversa indexes Hyperliquid's clearinghouse directly and computes analytics that are difficult to obtain from public web sources or generic blockchain APIs.

**Builder dex support:** 369+ markets across 8 dexes, including commodities, stocks, indices, and perps.

**HIP-4 support:** outcome-contract discovery, question metadata, recent fills, settlements, daily volume, top outcome traders, wallet outcome history, outcome/perp trader overlap, and current open perp-position context for outcome holders.

---

## Safety Boundary: Read-Only Analytics Only

Coinversa Pulse is a market-data and analytics MCP server.

This skill does **not** expose any tools for:

- Trading
- Order placement
- Wallet signing
- Transaction signing
- Fund movement
- Token transfers
- Account approvals
- Hyperliquid agent wallet approval
- Backend signer approval
- Custody or control of assets
- Managing margin or leverage settings

No private key, seed phrase, wallet signature, exchange credential, or Hyperliquid account approval is required to use this skill.

Users should **not** approve a Hyperliquid agent wallet, backend signer, trading agent, or any account-level trading permission for this MCP skill. No such approval is needed for Coinversa Pulse.

If Coinversa offers trading or execution functionality through another product, app, or integration, that functionality is outside the scope of this MCP skill and should be reviewed separately.

---

## Data & Privacy

Coinversa Pulse sends MCP tool requests to Coinversa's hosted API at `https://api.coinversa.ai` by default.

Depending on the tool used, requests may include:

- Market symbols
- Wallet addresses
- Cohort names
- HIP-4 outcome IDs
- Time windows
- API-key-authenticated usage metadata
- Requested analytics parameters

Do not submit private, sensitive, or nonpublic information unless you are comfortable sending it to Coinversa's API.

Coinversa Pulse does not require private keys, seed phrases, wallet signatures, exchange credentials, or Hyperliquid account approvals.

For more details, review Coinversa's website, API documentation, and privacy terms.

---

## Setup

An API key is required for every tool.

Get a key at [coinversa.ai/developers](https://coinversa.ai/developers).

You can connect in two ways:

| Method | Endpoint / command | Best for |
|--------|--------------------|----------|
| Hosted Remote MCP | `https://mcp.coinversa.ai/mcp` | Remote MCP clients and custom connectors that support Streamable HTTP |
| Local stdio MCP | `npx -y @coinversaa/mcp-server@0.8.0` | Claude Desktop, Cursor, Claude Code, Codex, and local MCP clients |

Remote MCP clients should send the Coinversa key as either:

```text
Authorization: Bearer cvsa_...
X-API-Key: cvsa_...
```

### Hosted Remote MCP

Use this URL for remote MCP clients:

```text
https://mcp.coinversa.ai/mcp
```

The hosted endpoint uses Streamable HTTP. MCP requests without a Coinversa API key are rejected.

### Local stdio MCP

Use `npx` when the MCP client runs local stdio servers.

#### Claude Desktop

Edit:

```text
~/Library/Application Support/Claude/claude_desktop_config.json
```

Add:

```json
{
  "mcpServers": {
    "coinversaa": {
      "command": "npx",
      "args": ["-y", "@coinversaa/mcp-server@0.8.0"],
      "env": {
        "COINVERSAA_API_KEY": "cvsa_your_key_here"
      }
    }
  }
}
```

#### Cursor

Add to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "coinversaa": {
      "command": "npx",
      "args": ["-y", "@coinversaa/mcp-server@0.8.0"],
      "env": {
        "COINVERSAA_API_KEY": "cvsa_your_key_here"
      }
    }
  }
}
```

#### Claude Code

```bash
claude mcp add coinversaa -- npx -y @coinversaa/mcp-server@0.8.0
export COINVERSAA_API_KEY="cvsa_your_key_here"
```

#### OpenClaw

```bash
openclaw skill install coinversaa-pulse
```

---

## Access Tiers

All tools require an API key. Backend tiering is enforced by the Coinversa API.

| Tier | Typical access | Requests/min | Daily cap | Monthly cap |
|------|----------------|--------------|-----------|-------------|
| Free API key | Public discovery, market data, selected HIP-4 discovery routes | 30 | 1,000 | - |
| Starter | Free routes plus selected trader and HIP-4 analytics | 120 | 2,000 | 50,000 |
| Pro | Starter plus deeper risk, historical, official OI, and overlap analytics | 600 | 20,000 | 500,000 |
| Enterprise | Custom access and limits | Custom | Custom | Custom |

Rate-limit headers may include `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`, `X-RateLimit-Tier`, and `X-RateLimit-Daily-Remaining`.

---

## Builder Dex Markets


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Coinversaa/mcp-server](https://github.com/Coinversaa/mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
