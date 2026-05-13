---
trigger: always_on
description: > This file is loaded every turn. It is the highest-leverage configuration for any AI assistant working on this codebase.
---

# CLAUDE.md

> This file is loaded every turn. It is the highest-leverage configuration for any AI assistant working on this codebase.

---

## Architecture — The Big Picture

### Three brands, three repos

```
t2000 (this repo)    → Infrastructure: CLI, SDK, MCP, engine, gateway, contracts
audric (separate)    → Consumer product: audric.ai website, app, Chrome extension
suimpp (separate)    → Protocol: suimpp.dev, @suimpp/mpp, @suimpp/discovery
```

### This repo structure

```
t2000/
├── apps/gateway     ← MPP API gateway (mpp.t2000.ai, 40+ services, 88 endpoints)
├── apps/server      ← Backend API
├── apps/web         ← t2000.ai marketing website
├── packages/cli     ← @t2000/cli (npm)
├── packages/sdk     ← @t2000/sdk (npm)
├── packages/engine  ← @t2000/engine (agent engine — QueryEngine, tools, MCP)
├── packages/mcp     ← @t2000/mcp (npm)
├── t2000-skills/    ← Agent skill definitions
└── audric-roadmap.md ← Product roadmap + build tracker
```

### Two brand layers

**t2000** = infra. Names the underlying capabilities (engine, SDK, MCP, MPP gateway, contracts). Used in technical docs, package names, READMEs, dev-facing surfaces.

**Audric** = consumer. Names the surfaces a user touches. Always one of exactly **five products** (post-S.18 reframe): **Audric Passport, Audric Intelligence, Audric Finance, Audric Pay, Audric Store**. (S.17's 4-product cut overloaded Intelligence as both "moat" and "home for every financial verb." S.18 splits Finance back out — it's the home for save / borrow / swap / charts. Send + Receive collapse into Pay.)

#### The five products

| Audric product | What it is | t2000 layer |
|---|---|---|
| 🪪 **Audric Passport** | The trust layer. Identity (zkLogin via Google), non-custodial wallet on Sui, tap-to-confirm consent on every write, sponsored gas. Wraps every other product. | `@t2000/sdk` (wallet, signing) + Enoki (zkLogin, gas sponsorship) + `@mysten/sui` |
| 🧠 **Audric Intelligence** | The brain (the moat). Five systems orchestrate every money decision — Agent Harness (34 tools), Reasoning Engine (14 guards, 6 skill recipes), Silent Profile, Chain Memory, AdviceLog. Picks the tool, clears the guards, remembers what it told you. Engineering-facing brand; users experience it as "Audric just understood me." | `@t2000/engine` (QueryEngine + tools + reasoning + guards + skill recipes) |
| 💰 **Audric Finance** | Manage your money on Sui. Save (NAVI lend, 3–8% APY on USDC or USDsui — strategic exception added in v0.51.0), Credit (NAVI borrow USDC or USDsui against savings, health factor visible at all times — repay must use the same asset as the borrow), Swap (Cetus aggregator, best-route across 20+ DEXs, 0.1% fee), Charts (interactive yield / health / portfolio visualizations rendered from chat). Every write taps to confirm via Passport. | `@t2000/sdk` NAVI lending/borrowing builders + `cetus-swap.ts` + `@t2000/engine` chart canvas templates |
| 💸 **Audric Pay** | Move money. Free, global, instant on Sui. Send USDC to anyone, receive via payment links / invoices / QR. No bank, no borders, no fees. | `@t2000/sdk` Sui tx builders (direct USDC transfers, payment-link contract, invoice flows) |
| 🛒 **Audric Store** | Creator marketplace at `audric.ai/username`. Generate AI music, art, ebooks, list them, sell in USDC. 92% to creator. **Coming soon (Phase 5).** | `@t2000/sdk` + Walrus storage + payment links (built on Audric Pay primitives) |

#### Audric Passport — the trust layer (4 pillars)

> **Your passport to a new kind of finance.**

Every Audric action runs through Passport. It's the wallet itself.

| Pillar | Meaning |
|---|---|
| 🪪 **Identity** | Sign in with Google. Your Passport is a cryptographic wallet, created in 3 seconds. No seed phrase. Yours forever. (zkLogin + Enoki) |
| ✋ **You decide** | Audric never moves money on its own. Every Finance and Pay action — save, send, swap, borrow — waits on your tap-to-confirm. |
| 🔐 **Sponsored gas** | We pay the network fees so you don't need SUI to transact. Your USDC stays your USDC. (Enoki sponsorship) |
| ⛓️ **Yours** | Non-custodial. We cannot move your money. Every transaction is on Sui mainnet, verifiable by anyone, forever. |

#### Audric Intelligence — the 5-system moat (the differentiator)

> **Not a chatbot. A financial agent.** Five systems work together to understand your money, reason about decisions, and get smarter over time. Every action still waits on your Passport tap-to-confirm.

| System | What it does | Implementation |
|---|---|---|
| 🎛️ **Agent Harness** | 34 tools, one agent. The runtime that orchestrates Finance ops (save, swap, borrow, repay, charts), Pay ops (send, receive), and read tools (balances, DeFi positions, analytics) inside a single conversation. Parallel reads, serial writes under a transaction mutex. | `@t2000/engine` `QueryEngine` + 34 tools (23 read / 11 write) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mission69b/t2000](https://github.com/mission69b/t2000) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
