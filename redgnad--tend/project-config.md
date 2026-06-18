---
trigger: always_on
description: - **Bags Hackathon** — $1M prize pool, 100 winners ($10K-$100K each)
---

# Tend — AI-managed revenue and growth control plane for Bags creator tokens

## Hackathon Context
- **Bags Hackathon** — $1M prize pool, 100 winners ($10K-$100K each)
- **Deadline**: 2026-06-01 (rolling review, ship early)
- **Tracks**: Claude Skills + AI Agents
- **Judging**: Onchain performance (volume, active traders, market cap) + App traction (DAU, MRR, GitHub stars)
- **Rule**: "Ideas alone don't qualify. You must deploy a working product with real users and real transactions."
- **Judges/mentors**: Solana, Helius, Meteora, Privy, DFlow, Birdeye

## Positioning
Tend transforms Bags.fm fee-sharing into a programmable growth engine:
- Services capture a % of trading fees on-chain
- An AI agent analyzes token/market state, decides what to do, executes, logs its reasoning
- The dashboard shows inputs → decision → action → impact → tx link
- MCP server lets creators manage everything through Claude Desktop

**Tend is NOT "MCP server with bots". Tend IS an AI-managed growth control plane for Bags tokens.**

## Stack
- Monorepo: npm workspaces (`packages/shared`, `packages/mcp-server`, `packages/agent`, `packages/frontend`)
- MCP: `@modelcontextprotocol/sdk` (STDIO transport)
- Solana: `@solana/web3.js` + `@bagsfm/bags-sdk`
- Frontend: Next.js 15, Tailwind v4
- Agent: Node.js scheduler + `@anthropic-ai/sdk` for decisional AI

## Build
```bash
npm run build           # all packages
npm run build:shared    # shared types + SDK wrapper
npm run build:mcp       # MCP server
npm run dev:dashboard   # Next.js dev
npm run dev:agent       # Agent runtime
```

## Key Architecture
- `packages/shared/src/bags-client.ts` — All Bags SDK interactions, handles tx signing (partialSign for claims)
- `packages/shared/src/db/` — Drizzle schema + Neon Postgres client (opt-in via `DATABASE_URL`; state.json remains default until migration lands)
- `packages/mcp-server/src/services/orchestrator.ts` — Fee-share config management (on-chain first, state second)
- `packages/mcp-server/src/state/` — StateManager (persists to `~/.tend/state.json`, unified wallet pool)
- `packages/mcp-server/src/tools/` — 7 MCP tools (1 prompt, 1 resource) covering the creator workflow
- `packages/shared/src/squads-client.ts` — Squads v4 PDA derivation, ix builders, SpendingLimit state reader (mandatory custody on the payout path)
- `packages/agent/src/payout-executor.ts` — refuses any payout without a Squads ref (no legacy admin-transfer fallback)
- `packages/agent/src/treasury-health.ts` — surplus check across all campaigns; scheduler gates accrual + withdrawals when the shared admin wallet runs low
- `packages/agent/src/` — Buyback agent + fee claimer + scheduler

## Conventions
- All shared types in `@tend/shared`
- Never `console.log` in MCP server (STDIO) — use `console.error` for debug
- BPS = basis points (100 = 1%, 10000 = 100%)
- All amounts in lamports internally, format with `formatSol()` for display

---

## DECISION FRAMEWORK — Every feature must pass this test

Before building or recommending ANY feature, ask:

1. **Does it improve a hackathon KPI?** (volume, active traders, DAU, MRR, GitHub stars)
2. **Is it visible in under 60 seconds in a demo?**
3. **Does it increase Tend's credibility as a premium product?**
4. **Does it produce real on-chain signals, or just complexity?**

If the answer to all 4 is "no", **don't build it**.

Prefer **2 real agents** over 6 fictional ones.
Prefer **real metrics** over decorated dashboards.
Prefer **working end-to-end flows** over impressive architecture diagrams.

---

## STRICT RULES — DO NOT VIOLATE

### Rule 1: No feature claims without working code
- A service in `service-registry.ts` MUST NOT have `status: "available"` unless there is real, tested execution logic in `packages/agent/src/` or equivalent.
- If the code only does generic `claimFees()`, the service is `"coming-soon"`, not `"available"`.
- "Available" means: user can add it → agent autonomously executes the specific strategy → results are observable in dashboard AND on-chain.

### Rule 2: No unused dependencies
- Every dependency in every `package.json` MUST be imported somewhere in the source code of that package.
- Aspirational dependencies (planned but not yet used) → REMOVE from package.json. Add when the import exists.

### Rule 3: No hardcoded status in UI
- The frontend MUST NOT hardcode service names, statuses ("ACTIVE", "LIVE"), or metrics.
- Data must come from API/state. If unavailable, show loading/empty state — not a fake badge.

### Rule 4: One write path per entity
- ONE canonical way to add/remove a service. Other surfaces call into it, not reimplement it.
- Known violation to fix: orchestrator (MCP) vs `/api/services/add` vs `/api/services/prepare+submit` all write state differently.

### Rule 5: State consistency
- `walletPool` mutations MUST be persisted to `state.json` immediately via `save()`. No separate wallets.json.
- Services added via any surface (MCP, API, dashboard) must be readable from any other surface.
- If a multi-tx sequence partially fails, DO NOT persist local state.

### Rule 6: Metrics must be real
- `totalFeesClaimed`, `totalFeesEarned`, `actionsPerformed` MUST be updated by the code that actually performs claims/actions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RedGnad/Tend](https://github.com/RedGnad/Tend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
