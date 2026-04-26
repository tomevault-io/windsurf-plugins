---
trigger: always_on
description: > Complete reference for generating, explaining, and troubleshooting `revx` CLI commands.
---

# Revolut X CLI Skill (`revx`)

> Complete reference for generating, explaining, and troubleshooting `revx` CLI commands.

> **Auth auto-resolve.** If any `revx` command fails with an auth-related error — `Auth not configured`, `No private key found`, `Authentication failed (401)`, `Invalid signature`, `insecure permissions`, or a missing key/config file at `~/.config/revolut-x/` — apply the fix yourself before re-asking the user. Walk through the [Configuration](#configuration) and [Error Reference](#error-reference) sections of this document and run the appropriate fix command (`chmod 600 ~/.config/revolut-x/<file>`, `revx configure generate-keypair`, `revx configure set --api-key <key>`). Only escalate to the user for inputs only they can provide: the 64-character API key string, confirmation that they have registered the public key in their Revolut X profile, the keypair passphrase, or approval to overwrite an existing keypair. After the fix, retry the original command.

## Quick Reference

| Command Group | Purpose |
|---|---|
| `revx configure` | Setup API key, keypair, passkey |
| `revx account` | Balances |
| `revx market` | Currencies, pairs, tickers, candles, order book |
| `revx order` | Place, open, history, get, cancel, fills |
| `revx trade` | Private and public trade history |
| `revx monitor` | Live price/indicator alerts (10 types) |
| `revx strategy grid` | Backtest, optimize, run grid bot |
| `revx connector telegram` | Telegram notification management |
| `revx events` | View alert/notification events |

All data commands support `--json` or `--output json` for machine-readable output.

---

## Installation

### Prerequisites

- **Node.js >= 20** (check with `node -v`)
- **npm** (comes with Node.js)

### Install

```bash
npm install -g @revolut/revolut-x-cli && npm link @revolut/revolut-x-cli
```

After install, `revx` is available as a global command:

```bash
revx --version                # Should print the version
```

---

## User Journey: From Install to First Trade

### Step 1: Configure Authentication

```bash
revx configure                 # Interactive setup wizard
```

This will:
1. Generate an Ed25519 keypair (private + public key)
2. Display your public key — copy it
3. Prompt you to register the public key at **exchange.revolut.com -> Profile -> API Keys**
4. Create a new API key — tick the **"Allow usage via Revolut X MCP and CLI"** checkbox
5. Prompt for the 64-character API key you receive after registration

Or do it step-by-step:

```bash
revx configure generate-keypair          # Creates Ed25519 keypair
# Register public key at exchange.revolut.com -> Profile -> API Keys
# Create API key — tick "Allow usage via Revolut X MCP and CLI"
revx configure set --api-key <64-char-key>
```

> **Before trading:** review the security policy — <https://github.com/revolut-engineering/revolut-x-api/security>. Key precautions: run the CLI in a sandbox or dedicated OS user account, use per-purpose API keys (label agent-driven keys "agentic"), keep credential file permissions at `0o600`, and password-protect the private key PEM.

### Step 2: Verify Configuration

```bash
revx configure get             # Show config status (keys redacted)
revx configure path            # Print config directory path
```

### Step 3: (Optional) Set a Passkey

A passkey is required for placing and cancelling orders. Set it once:

```bash
revx configure passkey set     # Prompts for passkey
revx configure passkey status  # Verify passkey is set
```

### Step 4: Check Your Account

```bash
revx account balances          # View non-zero balances
```

### Step 5: Explore the Market

```bash
revx market tickers            # See all prices
revx market ticker BTC-USD     # Check a specific pair
revx market candles BTC-USD    # View recent price history
```

### Step 6: Place Your First Order

```bash
# Start small — buy $10 of BTC at market price
revx order place BTC-USD buy --quote 10 --market

# Check it
revx order history --symbols BTC-USD
```

### Step 7: Set Up Monitoring (Optional)

```bash
# Get Telegram alerts
revx connector telegram add --token <bot-token> --chat-id <chat-id> --test

# Monitor BTC price
revx monitor price BTC-USD --direction above --threshold 100000
```

### Step 8: Try a Grid Bot (Optional)

```bash
# Backtest first
revx strategy grid backtest BTC-USD --investment 500 --levels 10 --range 5

# Dry run (no real orders)
revx strategy grid run BTC-USD --investment 500 --levels 10 --range 5 --dry-run
```

---

## Configuration

### Config Commands

```bash
revx configure                          # Interactive setup wizard
revx configure get                      # Show config status (keys redacted)
revx configure set --api-key <key>      # Set API key
revx configure generate-keypair         # Generate Ed25519 keypair
revx configure path                     # Print config directory path
revx configure passkey set              # Set or change passkey
revx configure passkey remove           # Remove passkey
revx configure passkey status           # Show passkey status
```

### Config Location

| Platform | Path |
|---|---|
| macOS/Linux | `~/.config/revolut-x/` |
| Windows | `%APPDATA%\revolut-x\` |
| Override | `REVOLUTX_CONFIG_DIR` env var |

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [revolut-engineering/revolut-x-api](https://github.com/revolut-engineering/revolut-x-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
