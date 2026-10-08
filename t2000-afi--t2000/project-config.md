---
trigger: always_on
description: > This file is loaded every turn. It is the highest-leverage configuration for any AI assistant working on this codebase.
---

# CLAUDE.md

> This file is loaded every turn. It is the highest-leverage configuration for any AI assistant working on this codebase.

---

## Architecture — The Big Picture

### What this monorepo is (product first)

**t2000 the product** = the **open marketplace** (hire · work · earn in USDC) at
`t2000.ai` + Passport Connect at `mcp.t2000.ai`. **Stage: traction / live mainnet.**

**This repo** ships the open rails: `@t2000/{cli,sdk,id,serve,sui-x402,discovery}`,
Move contracts (`agent_id`, `a2a_escrow` + reputation), and Mintlify docs.
The **Next.js apps** that render the marketplace UI and host Connect deploy from
the **audric** monorepo — hosting split, **same product**, not “infra without a
storefront.” Audric the *brand* is also AI you can put to work at `audric.ai`
(credit for chat/inference; USDC on Passport for the marketplace).

Do not describe this repo to outsiders as “CLI/SDK only” or “stage unclear.”
Public pitch copy: root `README.md` · `PRODUCT.md` · `brandkit/VOICE.md`.

### Two brands / repos

```
t2000 (this repo)  → Marketplace rails: CLI, SDK, contracts, docs (+ PRODUCT.md)
audric (separate)  → Hosts t2000.ai + mcp.t2000.ai apps; also Audric AI (audric.ai)
```

### This repo structure

```
t2000/
├── apps/docs        ← Mintlify developer docs (docs.t2000.ai)
├── packages/cli     ← @t2000/cli (npm)
├── packages/sdk     ← @t2000/sdk (npm)
├── packages/id      ← @t2000/id (npm)    ← Agent ID — agent_id::registry client
├── packages/serve   ← @t2000/serve (npm) ← Merchant-side x402 router
├── packages/x402    ← @t2000/sui-x402 (npm) ← x402 dialect for Sui
├── packages/discovery ← @t2000/discovery (npm) ← x402 probe + OpenAPI extract
├── contracts/       ← Move (agent_id, a2a_escrow, …)
├── t2000-skills/    ← Agent skill definitions
└── PRODUCT.md       ← Product map SSOT
```

> Planned package names (`store`, `models`) may appear in internal roadmaps —
> they are **not** shipped and must not be described as half-built product.

### Two brand layers

**t2000** = open marketplace + economy rails (SDK, CLI, serve, sui-x402, Agent ID,
contracts). Names the marketplace (`t2000.ai`) and Passport Connect (`mcp.t2000.ai`).
(`@t2000/engine` was retired 2026-06-14.)

**Audric** = AI you can put to work at audric.ai (t2000 marketplace from chat;
private chat + Private Inference second) — and the deploy home for t2000’s
web/Connect apps.

#### Audric v3 — the current product (the canon)

> **AI you can put to work.** Put it on the t2000 marketplace — hire, claim,
> deliver, settle in USDC. Private chat and Private Inference are extra. Same
> Passport. Live at audric.ai (`audric/apps/web-v3`).

- **Models** — open/uncensored (Kimi, DeepSeek, Grok, GPT-OSS) + frontier (Claude, GPT-5.x, Gemini); **Auto** routing picks the model + reasoning effort + step budget per turn.
- **Agent** — live web search, visible cited multi-step research, and the t2000 marketplace operator: browse, hire, claim work, deliver, settle, sell, and pay Instant APIs from chat (auto-sign under limits; confirm when the host requires it; USDC on t2000).
- **Passport (wallet)** — non-custodial zkLogin wallet; send USDC + USDsui, free/instant/gasless; sponsored gas; spend gated by host limits / confirm UX.
- **Privacy** — zero data retention, encrypted private chats & files, decentralized memory on Walrus (opt-in, deletable). Second beat in public copy — not the H1.

**What changed from v2 (2026-06-14):** DeFi (NAVI save/borrow) removed and `@t2000/engine` retired — v3 composes the AI SDK directly over `@t2000/sdk` (on **AI SDK 7** as of 2026-06-25). The SDK write surface is now **send · swap (Cetus) · pay (x402)**.

For Audric product detail, see **`audric/CLAUDE.md`** + **`SPEC_AUDRIC_V3.md`** (the canon). The legacy v2 app (engine + NAVI) was **deleted from the audric repo 2026-07-24** — do not reintroduce its concepts. Published `@t2000/engine@4.x` remains on npm for historical consumers only.

### SDK write surface

**send** (gasless USDC/USDsui) · **swap** (Cetus) · **pay** (x402). That's all of it.
DeFi (NAVI lending: save/withdraw/borrow/repay/claim) was removed 2026-06-14 (S.444) —
if it ever returns, use MCP reads + thin `@mysten/sui` builders, never
`@naviprotocol/lending` or `@suilend/sdk`.

**Exception:** `@cetusprotocol/aggregator-sdk` is allowed for swap execution — multi-DEX routing across 20+ DEXs cannot be feasibly replaced by thin tx builders. All usage is isolated to `packages/sdk/src/protocols/cetus-swap.ts`.

---

## Critical Rules

1. **Don't reintroduce DeFi or an engine package.** `save` / `borrow` / Invest left the SDK 2026-06-14 (S.444); `@t2000/engine` was retired and deleted (S.442) — host apps compose the AI SDK directly over `@t2000/sdk`, and transaction-safety guards are agent-loop guards owned by the host, not the SDK. The stack is 6 packages: `@t2000/{sdk,cli,id,serve,sui-x402,discovery}` (dialect npm name is `sui-x402`; dir stays `packages/x402`; the deprecated stdio `@t2000/mcp` was deleted from the monorepo 2026-08-03 — Connect is the MCP surface). Lineage: `git log` + `SPEC_AUDRIC_V3.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [t2000-afi/t2000](https://github.com/t2000-afi/t2000) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
