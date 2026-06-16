---
trigger: always_on
description: ██████╗  ██████╗ ██╗      █████╗ ██████╗ ███████╗
---

# 33Labs CCA Vulnerability Scanner

```
 ██████╗  ██████╗ ██╗      █████╗ ██████╗ ███████╗
 ╚════██╗ ╚════██╗██║     ██╔══██╗██╔══██╗██╔════╝
  █████╔╝  █████╔╝██║     ███████║██████╔╝███████╗
  ╚═══██╗  ╚═══██╗██║     ██╔══██║██╔══██╗╚════██║
 ██████╔╝ ██████╔╝███████╗██║  ██║██████╔╝███████║
 ╚═════╝  ╚═════╝ ╚══════╝╚═╝  ╚═╝╚═════╝ ╚══════╝

  ██████╗ ██████╗  █████╗      █████╗  ██████╗ ███████╗███╗   ██╗████████╗
 ██╔════╝██╔════╝ ██╔══██╗    ██╔══██╗██╔════╝ ██╔════╝████╗  ██║╚══██╔══╝
 ██║     ██║      ███████║    ███████║██║  ███╗█████╗  ██╔██╗ ██║   ██║
 ██║     ██║      ██╔══██║    ██╔══██║██║   ██║██╔══╝  ██║╚██╗██║   ██║
 ╚██████╗╚██████╗ ██║  ██║    ██║  ██║╚██████╔╝███████╗██║ ╚████║   ██║
  ╚═════╝ ╚═════╝╚═╝  ╚═╝    ╚═╝  ╚═╝ ╚═════╝ ╚══════╝╚═╝  ╚═══╝   ╚═╝

 ◈ Single-pass audit engine for Uniswap CCA
 ◈ 9 core vectors ∙ 6 integration vectors
```

When the user asks to scan for CCA vulnerabilities or run a CCA audit, follow this workflow against all `.sol` files in the project (excluding `test/`, `script/`, `lib/`).

## Workflow

### Step 1 — Read
Read all in-scope `.sol` files. Prioritize main contracts first, then libraries and interfaces.

### Step 2 — Triage
For each of the 14 vectors below (VC1-VC8 core, VI1-VI6 integration), classify as Skip / Borderline / Survive. Output all tiers.

### Step 3 — Deep Analysis
Only surviving vectors. Trace call chain from entry point to vulnerable line. Check all guards. Apply FP Gate. DROP if any check fails, CONFIRM if all pass.

### Step 4 — Adversarial Pass
Reason freely about the code beyond the checklist. Focus on: logic errors in clearing price computation, unsafe external interactions, access control gaps, economic exploits (MEV, sandwich), integration footguns (stale state reads), arithmetic edge cases (Q96 overflow), DoS vectors.

For integrations: What if a third party calls claimTokens first? What if CCA state changes between read and use? What if auction params are malicious? What if a searcher front/back-runs transactions?

**Silent misconfigurations** (no attacker required — missing validation that silently produces wrong results. Nothing reverts, nothing breaks. The math just quietly gives wrong answers for a class of inputs nobody rejects):
- No decimal floor check on auction token: tokens below 6 decimals lose significant value to Q96 rounding. A 2-decimal token can lose 90%+ per operation. Auction proceeds normally — just silently misallocates.
- No minimum tickSpacing enforcement: deploying with tiny tick spacing enables gas-exhaustion DoS on every checkpoint. CCA docs recommend "at least 1 basis point of floor price" — guidance only, not enforced on-chain.
- No minimum mps on final auction step: near-zero last-step issuance makes final clearing price trivially manipulable.
- No bounds on floorPrice relative to tick extremes: extreme values create auctions where math works but economics are broken.

### Step 5 — Report
Summary header (files, lines, finding count by severity), then findings sorted by confidence.

## Core Vectors (CCA's own code)

**VC1 — TICK-ITERATION-DOS**: _iterateOverTicksAndFindClearingPrice walks linked list of ticks per checkpoint. Attacker initializes thousands at minimum spacing → gas exhaustion on every submitBid. CONFIRM IF: no gas-bounded exit or no minimum tick spacing.

**VC2 — PRORATA-MEV**: Clearing tick fills pro-rata via on-chain accumulators. Searcher front-runs to dilute, back-runs exitPartiallyFilledBid same block. Flashloan-composable. CONFIRM IF: no anti-dilution and same-block exit allowed.

**VC3 — STEP-TAIL-MANIPULATION**: Final clearingPrice seeds v4 pool. Tiny last-step mps = almost no supply anchors price. Single bid near END_BLOCK shifts pool opening. CONFIRM IF: no minimum mps enforcement on final step.

**VC4 — DIRECT-TRANSFER-LOSS**: sweep functions use accounted values not balanceOf. Direct transfers permanently lost. CONFIRM IF: sweep uses accounted values.

**VC5 — OVERFLOW-BOUNDS**: TOTAL_SUPPLY * nextActiveTickPrice_ can approach uint256 max. Guard may fire late. CONFIRM IF: product can overflow before guard.

**VC6 — LOWDEC-FOT-SILENT-MISALLOCATION**: <6 decimal tokens silently misallocate (Q96 rounding loss). Fee-on-transfer tokens drift accounting. The bug is the ABSENCE of validation — the auction creation path accepts any token without checking decimals. CONFIRM IF: no decimal floor check in constructor/factory OR no balanceOf-delta verification on transfers. Either alone is sufficient.

**VC7 — PERMISSIONLESS-CLAIM-ZEROING**: claimTokens() permissionless, zeros tokensFilled permanently. #1 integration bug source. CONFIRM IF: no access control on claimTokens.

**VC8 — BID-LOCK-V1**: v1.0.0 factory (0x0000ccaDF55C911a2FbC0BB9d2942Aa77c6FAa1D) locks bids. CONFIRM IF: v1.0.0 address in codebase.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [33Audits/cca-audit-agent](https://github.com/33Audits/cca-audit-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
