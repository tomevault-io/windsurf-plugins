---
trigger: always_on
description: This is a **BingX OpenAPI skill collection** providing 22 skills for perpetual contract, spot, coin-margined, copy trading, account operations, announcements, and WebSocket real-time data streams: market data queries, trading operations, account management, wallet operations, copy trading, sub-account management, agent/affiliate management, official announcements, and WebSocket market/account data subscriptions.
---

# BingX AI Skills — Agent Instructions

This is a **BingX OpenAPI skill collection** providing 22 skills for perpetual contract, spot, coin-margined, copy trading, account operations, announcements, and WebSocket real-time data streams: market data queries, trading operations, account management, wallet operations, copy trading, sub-account management, agent/affiliate management, official announcements, and WebSocket market/account data subscriptions.

## Available Skills

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| bingx-swap-market | Perpetual futures market data | User asks for swap price, depth, K-lines, funding rate, contract info, ticker, open interest, or any perpetual futures market data queries |
| bingx-swap-trade | Perpetual futures trading | User wants to place swap orders, cancel swap orders, modify orders, set leverage, set margin mode, query swap orders, or any swap trading operations |
| bingx-swap-account | Perpetual futures account | User asks for swap balance, positions, assets, income history, funding history, commission rate, or swap account information |
| bingx-spot-market | Spot market data | User asks for spot price, spot order book, spot K-lines, spot recent trades, spot trading symbols, or any spot market data queries |
| bingx-spot-trade | Spot trading | User wants to place spot orders, cancel spot orders, query spot open orders or order history, view trade fills, manage OCO orders, or check spot trading commission rates |
| bingx-spot-account | Spot account management | User asks for spot balance, asset overview, or wants to transfer assets between accounts |
| bingx-spot-wallet | Spot wallet operations | User wants to deposit, withdraw, query deposit/withdrawal history, or check coin network configuration |
| bingx-coinm-market | Coin-M perpetual market data | User asks for coin-margined perpetual futures price, depth, K-lines, funding rate, or market statistics |
| bingx-coinm-trade | Coin-M perpetual trading | User wants to place/cancel coin-margined perpetual orders, set leverage/margin mode, or manage positions |
| bingx-copytrade-spot | Spot copy trading | User wants to manage spot copy trading, sell copy trade positions, or query spot copy trading profit |
| bingx-copytrade-swap | Swap copy trading | User wants to manage perpetual contract copy trading, close copy trade positions, or set TP/SL on copy trades |
| bingx-fund-account | Fund account management | User asks about main account funds, asset overview, or fund transfers between accounts |
| bingx-sub-account | Sub-account management | User wants to create/list sub-accounts, manage sub-account API keys, freeze/unfreeze accounts, or transfer assets between sub-accounts |
| bingx-agent | Agent/affiliate management | User asks about agent invited users, affiliate commissions, referral relationships, partner data, or broker commission reports |
| bingx-standard-trade | Standard contract trading | User asks about standard contract positions, standard futures order history, or standard contract balance |
| bingx-announcement | Official announcements | User asks about BingX announcements, notices, promotions, product updates, maintenance notices, listing/delisting, or funding rate notices |
| bingx-swap-ws-market | Swap WebSocket market data | User asks for real-time swap market data, live perpetual futures price feeds, streaming swap order books, WebSocket depth/trade/kline/ticker subscriptions for perpetual futures |
| bingx-swap-ws-account | Swap WebSocket account data | User asks for real-time swap account updates, live order status streaming, position change notifications, or WebSocket account monitoring for perpetual futures |
| bingx-spot-ws-market | Spot WebSocket market data | User asks for real-time spot market data, live spot price feeds, streaming spot order books, WebSocket depth/trade/kline/ticker subscriptions for spot trading |
| bingx-spot-ws-account | Spot WebSocket account data | User asks for real-time spot account updates, live spot order status streaming, spot balance change notifications, or WebSocket account monitoring for spot trading |
| bingx-coinm-ws-market | Coin-M WebSocket market data | User asks for real-time coin-margined futures market data, live Coin-M price feeds, streaming inverse futures order books, or WebSocket subscriptions for coin-margined perpetual futures |
| bingx-coinm-ws-account | Coin-M WebSocket account data | User asks for real-time Coin-M account updates, live coin-margined order status, inverse futures position changes, or WebSocket account monitoring for coin-margined perpetual futures |

## Skill Discovery

Skills are in the `skills/` directory. Each skill contains a `SKILL.md` with:

- YAML frontmatter (name, description, metadata)
- Full API reference with endpoints, parameters, and response schemas
- Code examples (TypeScript)
- Authentication implementation (HMAC SHA256)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BingX-API/api-ai-skills](https://github.com/BingX-API/api-ai-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
