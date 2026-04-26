---
trigger: always_on
description: BARK is an AI-powered meme token risk auditor purpose-built for Four.Meme on BNB Chain. It auto-scans every new token launch, scores rug pull risk using AI, and learns from scam patterns over time via persistent memory.
---

# BARK — Blockchain Audit & Risk Keeper

## Project Overview

BARK is an AI-powered meme token risk auditor purpose-built for Four.Meme on BNB Chain. It auto-scans every new token launch, scores rug pull risk using AI, and learns from scam patterns over time via persistent memory.

**Tagline:** "We bark before you get bitten."

**Hackathon:** Four.Meme AI Sprint (DoraHacks) — Deadline April 30, 2026
**Prize Pool:** $50,000 USD main + $2,000 Pieverse bounty + $3,000 dgrid credits

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  FRONTEND (Next.js + Tailwind)           │
│  Live Feed / Token Search / Risk Dashboard / Score Card  │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│              BARK API (TypeScript / Node.js)              │
│                                                          │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────┐   │
│  │ Contract     │  │ Holder       │  │ Social        │   │
│  │ Analyzer     │  │ Analyzer     │  │ Analyzer      │   │
│  │ (GoPlus API) │  │ (BSCScan)    │  │ (optional)    │   │
│  └──────┬──────┘  └──────┬───────┘  └──────┬────────┘   │
│         └────────────────┼──────────────────┘            │
│                          ▼                               │
│              ┌───────────────────┐                       │
│              │   RISK SCORING    │                       │
│              │   (dgrid AI LLM)  │                       │
│              │   Score: 0-100    │                       │
│              └────────┬──────────┘                       │
└───────────────────────┼──────────────────────────────────┘
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
┌──────────────┐ ┌────────────┐ ┌──────────────┐
│   Unibase    │ │  Pieverse   │ │  BSC         │
│   Membase    │ │  x402       │ │  Contracts   │
│   (memory)   │ │  (payments) │ │  (registry)  │
└──────────────┘ └────────────┘ └──────────────┘
```

## Tech Stack

| Layer          | Technology                                      |
| -------------- | ----------------------------------------------- |
| Frontend       | Next.js 14+ (App Router) + Tailwind CSS         |
| Backend/API    | TypeScript / Node.js                             |
| AI Inference   | dgrid AI Gateway (`https://api.dgrid.ai/v1`)     |
| Memory         | Unibase Membase SDK (Python/TS)                  |
| Identity       | ERC-8004 (Unibase agent identity)                |
| Payments       | Pieverse x402 + pieUSD (gasless)                 |
| Contracts      | Solidity + Foundry (BSC testnet)                 |
| Data Sources   | GoPlus API, BSCScan API, DexScreener API         |
| Font Stack     | Inter (body), Space Grotesk (headings/scores), JetBrains Mono (addresses) |

## Brand Identity

### Colors
```
Primary:
  Deep Navy      #0A1628   — background
  Electric Amber #F59E0B   — primary accent

Score Colors:
  Danger Red     #EF4444   — DANGER (0-39)
  Warning Orange #F97316   — CAUTION (40-69)
  Safe Green     #22C55E   — SAFE (70-100)

Neutral:
  Slate Gray     #94A3B8   — secondary text
  White          #F8FAFC   — primary text on dark bg

Gradient (hero/banner):
  Navy → Dark Purple  #0A1628 → #1E1B4B
```

### Design Direction
- Dark theme (AI/security tool convention)
- Clean, geometric, modern tech aesthetic
- NOT pixel art — professional security dashboard
- Think DexScreener / DEXTools aesthetic
- Mascot: Shiba Inu guard dog (alert, vigilant, not cutesy)

### Score Badge Design
```
🟢 SAFE    (70-100) — Green badge
🟡 CAUTION (40-69)  — Orange badge
🔴 DANGER  (0-39)   — Red badge
```

## Four.Meme Integration Details

### Contract
- **TokenManager2:** `0x5c952063c7fc8610FFDB798152D69F0B9550762b` (BSC mainnet)
- **TokenPurchase event:** signature hash `0x7db52723a3b2cdd6164364b3b766e65e540d7be48ffa89582956d8eaebe62942`
- All event parameters are **non-indexed** — must manually decode data field (each param = 32 bytes / 64 hex chars)

### Token Lifecycle on Four.Meme
1. Creator deploys token (fixed 10B supply, no custom functions, no taxes)
2. Token trades on internal bonding curve
3. At ~24 BNB accumulated → auto-graduates to PancakeSwap with liquidity pair
4. Post-graduation: trades on PancakeSwap (external market)

### Detection Approach
- Poll Four.Meme contract events for new token creations (every 10 seconds for MVP)
- Each new token → auto-trigger audit pipeline
- Can also use Bitquery APIs or BSCScan API for event monitoring

## Data Sources

### GoPlus Security API (FREE — primary contract analysis)
- Endpoint: `https://api.gopluslabs.io/api/v1/token_security/{chain_id}?contract_addresses={address}`
- Chain ID for BSC: `56`
- Returns: honeypot check, mint function, owner powers, proxy, blacklist, tax info, 30+ security checks
- Use as INPUT data — we add AI reasoning layer on top

### BSCScan API
- Deployer wallet history (previous contracts created)
- Top token holders
- Contract source/verification status

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EndPx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
