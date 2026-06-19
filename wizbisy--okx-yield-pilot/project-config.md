---
trigger: always_on
description: Autonomous multi-chain yield farming autopilot — auto-compound, rebalance, and risk-guard DeFi positions across Solana and X Layer using onchainOS. Do NOT use for: manual one-off swaps (okx-dex-swap), DeFi product discovery or deposit (okx-defi-invest), DeFi position viewing only (okx-defi-portfolio), wallet balance checks (okx-wallet-portfolio), token prices (okx-dex-market). Triggers (EN): 'auto-compound my yields', 'compound my DeFi rewards', 'rebalance my yield farms', 'optimize my DeFi port
---


# OKX Yield Pilot

Autonomous yield farming autopilot — auto-compound, portfolio rebalance, and risk-guard DeFi positions across Solana and X Layer via onchainOS CLI.

For CLI parameter details, see [references/cli-reference.md](references/cli-reference.md).

## Step 0 — Skill Routing (run before every other step)

Before running any workflow in this skill, classify the user's intent:

- **One-off DeFi deposit/withdraw** (no automation) → use `okx-defi-invest`
- **View DeFi positions only** → use `okx-defi-portfolio`
- **One-off token swap** → use `okx-dex-swap`
- **Wallet balance check** → use `okx-wallet-portfolio` or `okx-agentic-wallet`
- **Token price / chart** → use `okx-dex-market`
- **Named DApp interaction** (Aave, Raydium, Orca, etc.) → use `okx-dapp-discovery`

Stay in this skill when the user wants **ongoing automated management**: auto-compounding, scheduled rebalancing, portfolio optimization, IL monitoring, or performance reporting across yield farm positions.

## Skill Routing Table

| User intent                                          | Target skill           |
| ---------------------------------------------------- | ---------------------- |
| Auto-compound / harvest & reinvest / yield autopilot | **This skill**         |
| Rebalance DeFi allocations automatically             | **This skill**         |
| Monitor IL / APY anomalies / farming health          | **This skill**         |
| View farming performance report                      | **This skill**         |
| One-off DeFi deposit/withdraw (no automation)        | `okx-defi-invest`      |
| View current DeFi positions (no action)              | `okx-defi-portfolio`   |
| Swap tokens                                          | `okx-dex-swap`         |
| Check wallet balance                                 | `okx-wallet-portfolio` |
| Token price or chart                                 | `okx-dex-market`       |
| Named DApp (Aave, Orca, Raydium…)                    | `okx-dapp-discovery`   |

## Pre-flight Checks

> Read the preflight checks before any workflow. Look for the file in this order:
>
> 1. `_shared/preflight.md` (relative to this SKILL.md file)
> 2. `../_shared/preflight.md` (if skill is nested inside a parent skills folder)
> 3. `../okx-agentic-wallet/_shared/preflight.md` (shared OKX preflight)
>
> If none exist, run these checks inline: (a) `onchainos wallet status` — verify logged in, (b) `onchainos wallet addresses` — resolve EVM + Solana addresses, (c) `onchainos defi positions` — verify at least one active position exists.

## Decision Tree

When the user's request reaches this skill, follow this decision tree to select the correct workflow:

```
User request received
│
├─ Mentions "scan" / "show" / "list" / "what positions" / "what rewards"?
│  → Flow 1: Scan Positions
│
├─ Mentions "compound" / "harvest" / "reinvest" / "claim and reinvest" / "复投"?
│  ├─ Specifies a single chain or pool?
│  │  → Flow 2: Auto-Compound (single position)
│  └─ Says "all" or no filter?
│     → Flow 2: Auto-Compound (batch — all eligible positions)
│
├─ Mentions "rebalance" / "reallocate" / "drift" / "adjust weights" / "平衡"?
│  → Flow 3: Rebalance
│
├─ Mentions "health" / "check" / "IL" / "impermanent loss" / "risk" / "APY drop" / "safe" / "健康"?
│  → Flow 4: Health Check
│
├─ Mentions "report" / "performance" / "how much earned" / "effective APY" / "summary" / "报告"?
│  → Flow 5: Performance Report
│
├─ Mentions "schedule" / "autopilot" / "automate" / "daily" / "weekly" / "自动化"?
│  → Flow 6: Schedule Automation
│
└─ Unclear?
   → Run Flow 1 (Scan) first, then suggest next action based on results
```

## Reasoning Chain

Before executing any workflow, the agent MUST reason through these questions internally (do NOT show to user):

1. **Which chains does the user care about?** → If unspecified, scan ALL supported chains
2. **Are the positions worth acting on?** → Check reward value vs gas cost BEFORE executing
3. **Is there a safety concern?** → Run health gates mentally before suggesting actions
4. **What's the optimal execution order?** → Group operations by chain to minimize wallet auth overhead
5. **Can I save the user gas?** → Always prefer X Layer (zero gas) for intermediate operations

## onchainos Commands Used

> Full parameter tables, return schemas, and usage examples: [`references/cli-reference.md`](references/cli-reference.md)

Commands used across flows: `defi positions`, `defi position-detail`, `defi detail`, `defi collect`, `defi invest`, `defi withdraw`, `defi rate-chart`, `defi tvl-chart`, `swap quote`, `swap execute`, `wallet status`, `wallet addresses`, `wallet contract-call`, `portfolio all-balances`, `token search`, `token price-info`

## Chain Support


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Wizbisy/okx-yield-pilot](https://github.com/Wizbisy/okx-yield-pilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
