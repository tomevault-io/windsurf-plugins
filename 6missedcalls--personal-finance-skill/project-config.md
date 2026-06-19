---
trigger: always_on
description: >
---


# Personal Finance Skill

A comprehensive personal finance management skill with 75 tools across 7 extensions for banking, investing, tax, market intelligence, social sentiment, and financial analysis workflows.

## When to Use

Activate this skill when a user asks for:

- **Account aggregation** — connecting bank accounts, viewing balances, syncing transactions
- **Net worth / cash flow** — computing totals, tracking spending, savings rate analysis
- **Portfolio monitoring** — positions, allocation, performance, drift detection
- **Trading** — placing/canceling orders, market data, asset lookup (Alpaca)
- **Tax optimization** — estimated liability, TLH candidates, wash sale checks, quarterly payments
- **Tax document processing** — parsing W-2, 1099-B/DIV/INT, K-1, Form 1040, Schedules A-E/SE, Form 8949, Form 6251 (AMT), state returns
- **Market intelligence** — company news, SEC filings, analyst recommendations, economic data (FRED, BLS), news sentiment
- **Social sentiment** — StockTwits sentiment, X/Twitter cashtag analysis, trending symbols, congressional trading
- **Recurring expense tracking** — subscriptions, bills, income streams
- **Anomaly detection** — unusual transactions, balance drops, duplicate charges
- **Financial briefings** — weekly/monthly summaries with action items
- **Scheduled finance workflows** — cron-based scans, alerts, reports

## Architecture Overview

Seven extensions organized in three layers:

```
Intelligence Layer
  tax-engine (23 tools) — parsing (15), liability, TLH, wash sales, lots,
    Schedule D computation, state tax, AMT
  market-intel (10 tools) — news, fundamentals, SEC filings, economic data
  social-sentiment (6 tools) — StockTwits, X/Twitter, congressional trades

Data Source Adapters
  plaid-connect (8)   alpaca-trading (10)   ibkr-portfolio (9)

Foundation Layer
  finance-core (9 tools) — canonical models, storage, normalization,
    policy checks, anomaly detection, briefs
```

**Data flow**: Adapters fetch provider data → finance-core normalizes and stores → intelligence layer analyzes → policy engine gates actions.

## Tool Catalog

### finance-core — 9 tools

| Tool | Description | Risk |
|------|-------------|------|
| `finance_upsert_snapshot` | Store normalized financial data snapshot (idempotent) | LOW |
| `finance_get_state` | Get current financial state (accounts, positions, etc.) | READ |
| `finance_get_transactions` | Query transactions with filters and pagination | READ |
| `finance_get_net_worth` | Calculate net worth breakdown by category/account | READ |
| `finance_detect_anomalies` | Scan for unusual transactions, balance drops, fee spikes | READ |
| `finance_cash_flow_summary` | Income vs expenses by category with savings rate | READ |
| `finance_subscription_tracker` | Identify recurring charges and subscription patterns | READ |
| `finance_generate_brief` | Create structured financial summary with action items | READ |
| `finance_policy_check` | Validate proposed action against policy rules | READ |

> Full schemas: [references/ext-finance-core.md](references/ext-finance-core.md)

### plaid-connect — 8 tools

| Tool | Description | Risk |
|------|-------------|------|
| `plaid_create_link_token` | Initialize Plaid Link for account connection | LOW |
| `plaid_exchange_token` | Exchange public token for permanent access | MED |
| `plaid_get_accounts` | List connected accounts with balances | READ |
| `plaid_get_transactions` | Fetch transactions via cursor-based sync | READ |
| `plaid_get_investments` | Fetch holdings, securities, investment transactions | READ |
| `plaid_get_liabilities` | Fetch credit, student loan, and mortgage data | READ |
| `plaid_get_recurring` | Identify recurring inflow/outflow streams | READ |
| `plaid_webhook_handler` | Process incoming Plaid webhook events | LOW |

> Full schemas: [references/ext-plaid-connect.md](references/ext-plaid-connect.md)

### alpaca-trading — 10 tools

| Tool | Description | Risk |
|------|-------------|------|
| `alpaca_get_account` | Get account balances, buying power, status | READ |
| `alpaca_list_positions` | List all open positions | READ |
| `alpaca_get_position` | Get single position by symbol | READ |
| `alpaca_list_orders` | List orders with status/date filters | READ |
| `alpaca_create_order` | Submit buy/sell order with safety checks | **HIGH** |
| `alpaca_cancel_order` | Cancel a pending order | MED |
| `alpaca_portfolio_history` | Historical equity and P/L over time | READ |
| `alpaca_get_assets` | Search tradable assets by class/exchange | READ |
| `alpaca_market_data` | Get snapshots, bars, or quotes for symbols | READ |
| `alpaca_clock` | Check if market is open, next open/close | READ |

> Full schemas: [references/ext-alpaca-trading.md](references/ext-alpaca-trading.md)

### ibkr-portfolio — 9 tools

| Tool | Description | Risk |
|------|-------------|------|
| `ibkr_auth_status` | Check gateway authentication status | READ |
| `ibkr_tickle` | Keep gateway session alive (~1 min interval) | LOW |
| `ibkr_list_accounts` | List accounts (must call first) | READ |
| `ibkr_get_positions` | Get positions for an account (paginated) | READ |
| `ibkr_portfolio_allocation` | Allocation by asset class, sector, group | READ |
| `ibkr_portfolio_performance` | NAV time series and returns | READ |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [6missedcalls/personal-finance-skill](https://github.com/6missedcalls/personal-finance-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
