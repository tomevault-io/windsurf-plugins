---
trigger: always_on
description: Welcome to **Opencatz AI (Robinhood Chain Edition)**! This document outlines project conventions, tech stack, directory layout, and architectural rules for AI agents and developers working on this repository.
---

# AGENTS.md - Opencatz AI (Robinhood Chain Edition) Guidelines & Agent Instructions

Welcome to **Opencatz AI (Robinhood Chain Edition)**! This document outlines project conventions, tech stack, directory layout, and architectural rules for AI agents and developers working on this repository.

---

## 1. Project Overview

**Opencatz AI (Robinhood Chain Edition)** is an autonomous, multi-agent crypto intelligence and trading ecosystem specialized for **Robinhood Chain (EVM)** and operated through a **Discord Command Center**, **Terminal TUI**, and **Telegram Notification Bridge**.

- **Core Hub Agent (`#opencatz-control-room`):** Handles user chat, configuration, portfolio tracking, global risk management, custom price alerts (`/alert`), trade execution, and natural language trade audits.
- **Swarm Consensus Engine:** Evaluates candidate signals through a 3-Layer Filter (Quant & Liquidity, Catalyst & Sentiment, Security Audit) requiring a **>= 80% Confidence Score** before posting to Discord.
- **Specialist Screening Sub-Agents:** Run 24/7 background screening (on-demand) and post call signals to dedicated Discord channels (exactly 5 domains):
  - `#call-meme-robinhood` (Robinhood Chain EVM DEX tokens / GMGN OpenAPI + GoPlus security)
  - `#call-lp-robinhood` (Robinhood Chain Concentrated Liquidity Velocity Signals / Krystal Cloud)
  - `#call-nft-robinhood` (EVM NFT floor & rarity alerts / OpenSea REST v2)
  - `#call-alpha-robinhood` (1-hour Robinhood Chain Alpha Scraper / optional official X API v2)
  - `#call-whale-eth` (Hyperliquid ETH whale positioning & spot flow tracking)
- **Position Manager:** Handles post-execution auto-sell targets (Take Profit, Stop Loss, Trailing Stops, and Out-of-Range LP Warnings).

---

## 2. Technology Stack & Environment

- **Runtime:** Node.js (>=22.12) / TypeScript
- **Config:** `dotenv` (.env files, never committed)
- **Discord Bot SDK:** `discord.js` (v14+)
- **Target Chain:** Robinhood Chain (EVM L2) — chain ID **4663**, native token **ETH**, canonical RPC `https://rpc.mainnet.chain.robinhood.com`, explorer `https://robinhoodchain.blockscout.com`
- **Primary DEX Venue:** Uniswap V3 Router (Robinhood Chain EVM L2 #4663) — primary venue for meme tokens, swaps, and LP positions. Single-chain focus (cross-chain bridge removed).
- **Execution Modes (`EXECUTION_MODE`):**
  - `AUTO_EXECUTE`: Real on-chain trading via Uniswap V3 / Viem client when Swarm Consensus $\ge 80\%$ and Risk Manager checks pass. Requires `EVM_PRIVATE_KEY`.
  - `DRY_RUN`: Realistic market simulation using real-time quotes, fees, and price data from Uniswap V3 API / DexScreener. Requires public `EVM_WALLET_ADDRESS` (Private Key optional). Fills logged to `database/opencatz_state.json`.
  - `SIGNAL_ONLY`: Intelligence Hub mode posting screening call cards to Discord and auto-tracking wallet position holdings via `EVM_WALLET_ADDRESS`.
- **Blockchain & Crypto Web3 SDKs:**
  - `viem` (EVM reads/signs)
  - Uniswap V3 Gateway API (`https://trade-api.gateway.uniswap.org/v1`)
  - GMGN OpenAPI (smart-money / rank / trenches / token security audit)
  - Krystal Cloud DeFi Data API (Robinhood LP pools, `ethereum@4663`)
  - OpenSea REST API v2 (EVM NFTs + swap aggregator)
  - Relay.link (token send & secondary swap fallback)
- **Security Audit APIs:** GoPlus Security API (EVM) + GMGN `/v1/token/security`
- **AI Engine:** OpenRouter / OpenAI / Anthropic Node SDK
- **Database & State:** Local JSON file persistence (`database/opencatz_state.json`)
- **Protocol:** Model Context Protocol (MCP)

---

## 3. Directory Layout (verified against `git ls-files` — real paths only)

```
Opencatz AI (Robinhood Chain)/
├── .agents/
│   ├── AGENTS.md                  # Project rules & coding guidelines
│   └── skills/                    # Opencatz-specific skills (swarm trading, gmgn)
├── src/
│   ├── index.ts                   # Bot initialization & client launcher
│   ├── config/config.ts           # Env/config validation & execution mode helpers
│   ├── orchestrator/              # OpenCatz Core Hub & Global Risk Engine
│   │   ├── hub.ts                 # OpenCatzHub: agent states, risk gate, on-demand passes
│   │   ├── risk-manager.ts        # Drawdown / position-size / correlation guards
│   │   ├── risk-engine-v2.ts      # Kill-switch circuit breaker (singleton)
│   │   ├── swarm-consensus.ts     # 3-Layer Signal Quality Filter Engine
│   │   ├── swarm-learning.ts      # Outcome-driven agent weight recalibration
│   │   ├── strategy-engine.ts     # Sandboxed .mjs strategy loader (sanitized env)
│   │   ├── strategy-types.ts      # Strategy context types (snake_case GMGN contract)
│   │   ├── strategy-bootstrap.ts  # Preset/custom strategy selection + first-boot compile
│   │   ├── agent-registry.ts      # Single source of truth for all 5 agent domains
│   │   ├── agent-runner.ts        # LLM tool-call loop for chat/TUI/Telegram
│   │   ├── dispatch.ts            # Per-domain dispatch + LP payload builder
│   │   └── tool-registry.ts       # LLM function-calling tools (chat commands)
│   ├── agents/                    # Specialized screening agents (shared contract)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [muratmula/ai-robinhood-chain](https://github.com/muratmula/ai-robinhood-chain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
