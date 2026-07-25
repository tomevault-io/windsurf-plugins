---
trigger: always_on
description: **Prompt:** [`SECURITY.md`](../SECURITY.md) (3-round methodology: systematic → economic → adversarial triager)
---

# [Gemini (Gemini 3)](https://gemini.google.com/) — Moloch.sol

**Prompt:** [`SECURITY.md`](../SECURITY.md) (3-round methodology: systematic → economic → adversarial triager)
**Type:** AI audit guided by accumulated methodology from 15 prior audit tools
**Runs:** Two passes — Pass 1 received partial source (Shares, Loot, Badges, Summoner, utilities only); Pass 2 received full source

## Review Summary

> **Reviewed 2026-03-11. No valid findings across either pass.**
>
> - **Pass 1 (partial scope)** received only the auxiliary contracts (~half the file). The core Moloch contract — governance, voting, execution, ragequit, futarchy, sales, permits, multicall — was not provided. LOW-01 is a false positive and INFO-01 is marginal (see per-finding reviews below).
> - **Pass 2 (full scope)** received the complete source but produced a shallow architectural overview rather than a security audit. It identifies 5 items (H-01 through L-02), all of which are either known findings, design choices, or deployment constraints — zero novel findings. No line references, no attack paths, no adversarial validation. The report does not follow the SECURITY.md methodology despite being prompted with it.
> - **LOW-01 is a false positive (95% → 0%).** The initialization IS atomic: `Moloch.init()` (lines 235-240) deploys each clone via CREATE2 and calls `.init()` on the very next line, all within a single transaction. There is no mempool window. Gemini explicitly noted it couldn't see this code ("the logic for how dao links to its specific Loot and Badges instances is not visible in the Summoner snippet provided") but then assigned 95% confidence — contradicting its own uncertainty.
> - **INFO-01 / M-01 (badge gas griefing)** appears in both passes. Valid but marginal — `_recomputeMin` is O(populated_slots), bounded at 256. On mainnet ~50k gas; negligible on L2s. The gas is borne by whoever triggers the share balance change, not by the minSlot holder.
> - **H-01 (`delegatecall` arbitrary execution)** is KF#14 — a design choice, not a vulnerability. Equivalent to upgradeability.
> - **M-02 (EIP-1153 chain compatibility)** is a deployment constraint, not a security finding.
> - **L-01 (remainder to last in `_targetAlloc`)** is standard rounding behavior, not a vulnerability. Already verified by multiple prior audits.
> - **L-02 (`transfersLocked` centralization)** is an intentional DAO governance feature, not a vulnerability. Falls under the privileged-role rule.

---

# Pass 1: Partial-Scope Review (Shares, Loot, Badges, Summoner)

> This pass received only the auxiliary contracts. The core Moloch contract (lines 1-1000) was not provided. The report follows the SECURITY.md methodology but most category conclusions are based on code the model couldn't see.

## Executive Summary

* **Total findings:** 2 (Novel: 2, Duplicate: 0)
* **Critical:** 0 | **High:** 0 | **Medium:** 0 | **Low:** 1 | **Informational:** 1
* **Highest-confidence finding:** `Loot`/`Badges` Initialization Front-running at 95% confidence.

---

## Round 1: Systematic Code Review

### 1. Reentrancy

**Conclusion:** No issues found.
**Defense:** Within the provided scope (`Shares`, `Loot`, `Badges`), state changes occur before or without external calls to untrusted contracts. `Loot` and `Badges` only interact with the `DAO` address. The `Shares` contract uses internal checkpointing logic that does not hand over control flow to `msg.sender`.

### 2. Flash Loan / Vote Manipulation

**Conclusion:** No issues found.
**Defense:** The `_checkpointsLookup` function correctly utilizes a `blockNumber` parameter, which the architecture context confirms is set to `block.number - 1` during proposal creation. This ensures that any shares minted or transferred via `buyShares` or `transfer` in the same block as a proposal cannot be used to vote.

### 3. Governance Logic

**Conclusion:** No issues found (contextual).
**Defense:** The `Shares` delegation logic correctly uses `_repointVotesForHolder` to update voting power when splits are changed, preventing the "double-voting" or "lost-voting" errors common in custom delegation implementations.

### 4. Economic / Ragequit

**Conclusion:** No issues found in provided snippets.
**Defense:** The `Loot` contract correctly handles `burnFromMoloch`, ensuring the `totalSupply` and `balanceOf` are updated atomically.

### 5. Futarchy

**Conclusion:** No issues found in provided snippets.
**Defense:** The `Badges` contract correctly reads the `renderer` from the `DAO` to determine `tokenURI`, ensuring metadata is governance-controlled.

### 6. Access Control

**Conclusion:** **Finding [LOW-01]** identified regarding initialization.
**Defense:** While `onlyDAO` is robustly applied to functional methods, the initialization of auxiliary contracts (`Loot`, `Badges`) relies on a null-check that can be front-run.

### 7. Token Sales

**Conclusion:** No issues found in provided snippets.
**Defense:** `Loot` minting is strictly gated by `onlyDAO`.

### 8. Math / Precision

**Conclusion:** No issues found.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [z0r0z/majeur](https://github.com/z0r0z/majeur) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
