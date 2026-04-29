---
trigger: always_on
description: > Machine-readable instructions for coding agents working with ProxyGate.
---

# AGENTS.md — ProxyGate for AI Agents

> Machine-readable instructions for coding agents working with ProxyGate.
> Works with Claude Code, Codex CLI, Gemini CLI, Cursor, and any agent that reads AGENTS.md.

---

## What is ProxyGate?

ProxyGate is a marketplace where sellers list unused API capacity and AI agents purchase access through a transparent proxy. Keys never leave the server. Agents pay with USDC on Solana.

## CLI Quick Reference

```bash
npm install -g @proxygate/cli
proxygate login                 # Interactive setup (API key or wallet)
proxygate login --key pg_live_... # Direct API key auth
```

All commands support `--json` for structured machine-readable output and `--no-color` to disable ANSI.

---

## Commands

### Setup & Auth

```bash
proxygate login                              # Interactive menu (API key or wallet)
proxygate login --key pg_live_...            # API key auth (for agents)
proxygate login --keypair ~/.solana/id.json  # Wallet keypair auth
proxygate login --generate                   # Generate new wallet
proxygate whoami                             # Check auth mode + balance
proxygate logout                             # Remove API key
proxygate logout --all                       # Remove all auth
```

### Browse & Discover APIs

```bash
proxygate apis -q "weather" --json           # Search APIs by name/description
proxygate search geocoding --json            # Alias for apis -q
proxygate apis -s weather-api --json              # Filter by exact service slug
proxygate categories --json                  # List API categories
```

**Output schema (`--json`):**
```json
{
  "apis": [{
    "listing_id": "uuid",
    "service_slug": "string",
    "service_name": "string",
    "base_url": "string",
    "price_per_request": 10000,
    "pricing_unit": "per_request | per_token",
    "available_rpm": 60,
    "has_docs": true,
    "trust_score": 85,
    "listing_type": "proxy | tunnel | skill | product | dataset | service | connector"
  }]
}
```

### Make API Requests (Buy)

Use **service name**, slug, or listing UUID — the CLI resolves automatically:

```bash
# Proxy by service name (easiest)
proxygate proxy agent-postal-lookup /nl/1012

# POST with body
proxygate proxy weather-api /v1/forecast \
  -d '{"latitude":52.37,"longitude":4.90,"hourly":"temperature_2m"}'

# Stream a response (SSE)
proxygate proxy weather-api /v1/forecast --stream \
  -d '{"latitude":52.37,"longitude":4.90,"hourly":"temperature_2m"}'
```

**Output:** Raw upstream API response on stdout. Cost + request ID on stderr:
```
cost: $0.0155 | request: 905b1a53
```

**Exit codes:**
| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | CLI error (bad args, missing config) |
| 2 | Auth error (invalid keypair, nonce failure) |
| 3 | Insufficient credits |
| 4 | Upstream error (5xx, timeout) |
| 5 | Rate limited (retry after delay) |

### Balance & Payments

```bash
proxygate balance --json                     # Check USDC balance
proxygate deposit <amount> --json            # Deposit USDC
proxygate withdraw <amount> --json           # Withdraw USDC
```

**Balance output schema:**
```json
{
  "balance": 8750000,
  "pending_settlement": 98500,
  "available": 8651500,
  "currency": "lamports",
  "usdc_balance": 8.75,
  "usdc_available": 8.65
}
```

### Sell APIs

```bash
# Create a proxy listing (API with key)
proxygate listings create --non-interactive \
  --service-name "My API" \
  --base-url "https://api.example.com" \
  --auth-pattern bearer \
  --api-key "sk-..." \
  --categories "ai" \
  --price 10000 \
  --json

# Create a public API listing (no key)
proxygate listings create --non-interactive \
  --service-name "Weather API" \
  --base-url "https://api.open-meteo.com" \
  --auth-pattern none \
  --categories "weather" \
  --price 10000 \
  --json

# Create other listing types
proxygate listings create --non-interactive \
  --type skill \
  --service-name "Code Review" \
  --base-url "https://myskill.com" \
  --endpoint-url "https://myskill.com/invoke" \
  --categories "devtools" \
  --price 50000 \
  --json

# List your listings
proxygate listings list --json

# Upload API docs
proxygate listings upload-docs <listing-id> ./openapi.yaml

# Manage listings
proxygate listings update <id> --price 20000 --json
proxygate listings pause <id> --json
proxygate listings unpause <id> --json
proxygate listings delete <id> --yes --json
```

**Listing types:** `proxy` (default), `skill`, `product`, `dataset`, `service`, `connector`

**Type-specific flags:**
| Type | Required flag | Description |
|------|--------------|-------------|
| skill | `--endpoint-url <url>` | AI tool / MCP endpoint URL |
| product | `--file-url <url>` | Digital file download URL |
| dataset | `--bulk-url <url>` | Bulk data access URL |
| service | `--webhook-url <url>` | Webhook relay URL |
| connector | `--relay-url <url>` | Platform integration URL |

**Create output schema:**
```json
{
  "id": "uuid",
  "service": "slug",
  "is_active": true,
  "key_masked": "****5678",
  "sync_status": "synced"
}
```

### Tunnel (Expose Local Service)

```bash
proxygate tunnel                             # Start tunnel from proxygate.tunnel.yaml
proxygate tunnel --port 8080                 # Quick tunnel on port
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [proxygate-official/cli](https://github.com/proxygate-official/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
