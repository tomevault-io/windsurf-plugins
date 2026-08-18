---
trigger: always_on
description: This file is loaded automatically when this repo is opened in Claude Code. If a user
---

# Valet — guide for Claude Code

This file is loaded automatically when this repo is opened in Claude Code. If a user
asks you to help them set up or use Valet, follow this guide. Your role is twofold:
**help them install and connect Valet**, and — once it's running — **place the trades
they ask for**. The trading *decision* (what/when) is always the user's; Valet is just
the reliable execution plumbing.

Valet is a **monorepo of two MCP servers** over one shared safety core (`trading_core`):
`ibkr` — trades on **Interactive Brokers** (incl. **fractional shares by dollar amount**) —
and `crypto` — spot on **crypto exchanges** via CCXT. See `README.md` for the full picture
and `DECISIONS.md` for the reasoning behind the design. (The setup section below covers the
IBKR server; the crypto server has its own section further down.)

---

## Helping a user set it up

Walk the user through the steps below. Run the commands you can; clearly hand off the
ones only they can do (anything on the IBKR side — you cannot log in for them).

### 1. Install (you can do this)

```bash
python -m venv .venv
# Windows (PowerShell): & ".venv\Scripts\Activate.ps1"
# Linux/macOS:          source .venv/bin/activate
pip install -e ".[dev]"
cp .env.example .env
```

### 2. Configure `.env` (you can do this — ask the user for their values)

Set at least `IBKR_ACCOUNT_ID`. Keep the safe defaults: `IBKR_TRADING_MODE=paper`,
`TRADING_ALLOW_LIVE=false`, `TRADING_DRY_RUN=true`. Never commit `.env`.

### 3. The IBKR side (ONLY the user can do this — guide them clearly)

- An **IBKR Pro** account, open and funded (required by the API, even for paper).
- **Fractional permission**: Client Portal → Settings → Trading → Trading Permissions →
  Stocks → check **"Global (Trade in Fractions)"**.
- A **dedicated username** for the bot (IBKR allows one brokerage session per username;
  logging into TWS/mobile with the same user kills the gateway session).
- **Download and start the Client Portal Gateway** (Java app), then **log in via the
  browser** at `https://localhost:5000` with 2FA. This manual login is unavoidable —
  IBKR has no OAuth for retail. See the "Gateway setup" and "Login troubleshooting"
  sections of the README.

### 4. Register the MCP server with Claude Code

```bash
claude mcp add ibkr -- /path/to/.venv/Scripts/python.exe -m ibkr_agent.server.app
```

The tools appear in a **new** Claude Code session.

### 5. Verify

```bash
python -m ibkr_agent.healthcheck   # or: ibkr-healthcheck
```

A healthy result shows `authenticated=True connected=True`, the account flags
(`supportsCashQty`/`supportsFractions`), the balance, and a quote.

---

## Known issues

### Login goes through but nothing happens

**Symptom:** the user opens `https://localhost:5000`, logs in, approves 2FA, the page
loads — but then nothing happens. It just sits there and never reaches a logged-in
state, and the healthcheck keeps reporting `authenticated=False` / `connected=False`
(sometimes with `ssodh/init` returning HTTP 500 or a `no bridge` error).

**What fixes it:** restart the gateway cleanly and log in fresh. Stop the gateway's Java
process, start it again (`bin\run.bat root\conf.yaml` on Windows, `bin/run.sh
root/conf.yaml` on Linux/macOS), then reload `https://localhost:5000` and log in again.
A clean restart clears this in the large majority of cases — guide the user through it
first. Logging in from an incognito/private browser tab also helps (stale cookies can
get in the way).

**If it still persists:** it can also help to log out of any other IBKR session — IBKR
allows only one brokerage session per username, so a session open in IBKR Mobile or the
Client Portal web can block the gateway. Have the user log those out, restart the
gateway once more, and try again.

**Important — the login is not sticky.** Every time a fresh login is needed (the session
expired, the machine slept, the daily maintenance window passed), do the clean restart
*first*; don't just retry the login against the gateway that's already running. The
sequence is always: restart the gateway → then log in.

---

## Using Valet day to day

These are **all 20 MCP tools** you have once the `ibkr` server is connected — your full
capability surface. Every tool returns an `{"ok": bool, "data"/"error": ...}` envelope.

**Session & market**
- `session_status` — is the gateway authenticated/connected/competing, **and which
  account is live**: it returns `account_type` (`"LIVE"`/`"PAPER"`) from IBKR's
  `isPaper` (a LIVE account also returns a `warning`). This is the ground truth — the
  `IBKR_TRADING_MODE` label can disagree. Check it before trading so you never mistake
  a real-money account for paper. `portfolio` carries the same `account_type`.
- `market_status` — is the US market open (RTH) right now.

**Quotes & account (read-only)**
- `get_quote(symbol)` — last/bid/ask for one symbol.
- `get_quotes(symbols)` — quote a whole watchlist in **one** call (cheaper than N `get_quote`).
- `account_summary` — available funds, net liquidation, buying power.
- `positions` — open positions.
- `portfolio` — account summary + positions + total unrealized P&L in one snapshot.

**Before committing**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pedrobraiti/agentic-trading-mcp](https://github.com/pedrobraiti/agentic-trading-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
