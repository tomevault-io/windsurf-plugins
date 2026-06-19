---
trigger: always_on
description: Detect net-positive same-token arbitrage opportunities across EVM and Solana chains using only the onchainos CLI — price snapshot, live bridge fee (direct route or stablecoin-hop fallback), net P&L, and optional one-click execution.
---


# Cross-Chain Arbitrage Scanner

## Overview

Scans the same token across multiple chains, deducts the **live bridge fee** (direct
token↔token route when available, or a stablecoin-hop fallback when not) and
sell-leg gas estimate, and surfaces only **net-profitable** opportunities.

Most non-stablecoin tokens have no direct bridge route in OKX's aggregator
(Stargate / Across / Relay / Gas Zip pool USDC + native tokens, not LINK /
AAVE / UNI etc.). When the direct route fails, the skill automatically falls
back to a three-leg path: sell the token for USDC on the source chain, bridge
USDC, buy the token back on the destination chain.

Three behaviours:

1. **Signal mode (default)** — read-only report: price snapshot per chain,
   ranked routes by net P&L, explicit "no opportunity" message when every
   route is unprofitable.
2. **Execution mode (opt-in)** — when the user explicitly says "execute",
   runs the buy → bridge → sell legs through `onchainos swap execute` and
   `onchainos cross-chain execute`, with user confirmation gates.
3. **Refresh mode** — re-runs the scan with the same parameters, busting the
   30-second snapshot cache.

All numbers come from live `onchainos` CLI calls — no external APIs, no
fabricated or memorised prices.

### Trigger keywords

English: `cross-chain arbitrage`, `arbitrage scan`, `arb opportunity`,
`price gap across chains`, `which chain is ETH cheapest`,
`compare USDC price across chains`, `is there an arb on <symbol>`,
`bridge for profit`.

中文: `套利机会`, `跨链套利`, `跨链搬砖`, `哪条链上 ETH 最便宜`, `多链价差`,
`比较 USDC 在各链的价格`, `搬砖`.

### When NOT to trigger

| User said | Route to |
|---|---|
| "What's the price of ETH?" | `okx-dex-market` |
| "Bridge 100 USDC from Arbitrum to Base" | `okx-dex-bridge` |
| "Swap USDC for ETH on Base" | `okx-dex-swap` |
| "Find MEV / sandwich opportunities" | out of scope — explain politely |
| "CEX vs DEX arb" | out of scope — onchain-only |

A genuine arb intent requires (a) the same token and (b) ≥2 chains, or
asks "where is X cheapest". If only one chain is mentioned, ask the user
which destination chain(s) to compare against before scanning.

## Pre-flight Checks

Before using this skill, ensure:

1. The `onchainos` CLI is installed and on `PATH`. Install via:
   `npx skills add okx/onchainos-skills`. If `onchainos` is not found after
   install, run `export PATH="$HOME/.local/bin:$PATH"`.
2. Run `onchainos --version` once per session. If the upstream onchainos
   skill set is co-installed, follow its
   `okx-agentic-wallet/_shared/preflight.md` install/upgrade checks first.
3. For **execution mode only**: a logged-in OKX Agentic Wallet with native
   gas balance on the buy chain. Run `onchainos wallet status`; if not
   logged in, `onchainos wallet login`.

## Default scan universe

To keep latency low and protect API quota, defaults to the **top-5 EVM
chains** by stablecoin liquidity plus Solana if the token has a Solana CA:

| # | Chain | chainIndex |
|---|---|---|
| 1 | Ethereum | 1 |
| 2 | Arbitrum | 42161 |
| 3 | Base | 8453 |
| 4 | BNB Chain | 56 |
| 5 | Polygon | 137 |
| 6 | Solana (if applicable) | 501 |

The user can override with `chains: <list>`. Honour the user's list
exactly — do not silently expand or shrink.

## Tunable parameters

Defaults are tuned for the median arb-hunter use case ($1k notional,
mid-cap token, mainstream chains). The user can override any of them by
saying e.g. *"scan with 0.5% slippage and MEV protection off"*; honour
the override exactly.

| Param | Default | Notes |
|---|---|---|
| `slippage` | `0.01` (1%) | Applied to **both** `cross-chain quote` and `swap quote/execute`. ⚠️ **Unit mismatch in the CLI**: `cross-chain quote` takes a decimal fraction (`--slippage 0.01` = 1%) while `swap execute` takes a percent string (`--slippage 1` = 1%). When the user says "1%", translate per-CLI. |
| `mev` | `true` on Ethereum / BSC / Base (EVM chains where OKX supports MEV-protected routing); `false` elsewhere | Pass `--mev-protection` to every `swap execute` call on supported chains. On Solana, pair with `--tips 0.0001` (Jito tip in SOL). Other chains: silently omit; do not error. |
| `hop_stable` | Try `USDC` first; on failure (empty `dexRouterList` or `82000`) try `USDT` | See Step 4b/4c. The user can pin with `hop_stable: usdt` to skip USDC. |
| `priceImpact_max` | `1.5%` per DEX leg | Above this, route is tagged **high-impact** and de-prioritised in ranking. |
| `quote_timeout_s` | `8` | Soft per-call ceiling; see "Per-call soft timeout" note below. |

### Per-call soft timeout

The skill targets ~8 s per quote call. Implementation depends on shell
environment:

- **POSIX with GNU coreutils** (Linux distros, or macOS with `brew install
  coreutils`): wrap calls with `timeout 8s onchainos …` (Linux) or
  `gtimeout 8s onchainos …` (macOS).
- **macOS bare zsh / bash** without coreutils: do NOT use `timeout` — it
  is not installed by default and will fail. Rely on agent-orchestrated
  cancellation: pass the same per-call wall-time budget to the calling
  tool (the agent harness), or use `perl -e 'alarm(8); exec @ARGV' --

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jasonzhouu/cross-chain-arbitrage-skill](https://github.com/jasonzhouu/cross-chain-arbitrage-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
