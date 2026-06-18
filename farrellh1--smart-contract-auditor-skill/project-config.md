---
trigger: always_on
description: Systematic Solidity smart contract security audit using a 370-item checklist spanning 13 categories (attack patterns, DeFi, integrations, tokens, cross-chain, signatures, etc.). Use whenever the user asks to audit, review, find bugs in, or assess the security of Solidity/EVM smart contracts, or mentions solidit, slither findings, Code4rena/Sherlock/Cyfrin style review, reentrancy, oracle manipulation, access control review, or wants a checklist-driven review of a protocol. Trigger even when the 
---


# Smart Contract Security Audit

Drive a rigorous, checklist-anchored security review of Solidity/EVM smart contracts. The checklist is a curated distillation of real findings from Solodit (Code4rena, Sherlock, Cyfrin, Spearbit, etc.). Do not improvise a freestyle review — anchor every finding in an item id from the checklist so the user can trace your reasoning to prior art.

## Why checklist-driven

Ad-hoc reviews miss classes of bugs the reviewer hasn't personally seen before. The checklist encodes hundreds of real exploits by pattern. Walking it beats intuition, especially on categories adjacent to the reviewer's blind spots (e.g., a DeFi-native reviewer auditing a cross-chain bridge).

The full checklist is 370 items — too large to hold in working memory. It is split per category under `references/`. Load only the files relevant to the contract's scope. Do not preload everything.

## Workflow

### 1. Scope the contract

Before touching the checklist, read the source to answer:

- What does the protocol do in one sentence? (lending? AMM? staking? bridge? NFT mint? vault?)
- Which external systems does it touch? (Chainlink, Uniswap, LayerZero, ERC4626, Pendle, Aave, Curve, Balancer, GMX, Morpho, etc.)
- Which token standards? (ERC20 with fee-on-transfer? rebasing? ERC721? ERC1155? ERC4626?)
- Privileged roles and upgradeability pattern?
- Chains targeted (L1 only, L2, multi-chain)?
- Every `external` and `public` function — list them. Access-control gaps hide here.

**Protocol-specific pattern prompts (check during scoping):**

- **Lending/borrowing**: Does `liquidate()` require the liquidator to actually repay debt? Does the health-factor calculation normalize token decimals before comparing collateral value to debt value?
- **Oracle consumers**: Is `latestAnswer()` used instead of `latestRoundData()`? If so, there is no staleness check. Is the oracle return value cast from `int256` to `uint256` without a negative-value guard? Do oracle prices use 8 decimals while token amounts use 18 — is the mismatch handled?
- **Vaults / reward accounting**: Is pending reward zeroed when a user's balance changes before the pending amount is credited?
- **Privileged functions**: Does every admin function have an explicit `require(msg.sender == owner)` or role check? Is ownership transfer one-step (dangerous) or two-step?

State scoping findings back to the user in 3–6 bullets. This drives which reference files to load.

### 2. Pick the relevant reference files

Read `references/INDEX.md` first for the category map. Then load only what the scope justifies:

| Always load | For every audit |
|---|---|
| `attacker-s-mindset.md` | DOS, front-running, donation, sandwich, reentrancy framing |
| `basics.md` | Access control, arithmetic, storage, initialization, general Solidity pitfalls (largest file — 135 items) |
| `heuristics.md` | Smell-level patterns worth a second look |

| Load when scope matches | Trigger |
|---|---|
| `defi.md` | AMM, lending, vaults, yield, liquidations, oracle pricing, slippage |
| `token.md` | Custom ERC20/721/4626, fee-on-transfer, rebasing, hooks |
| `integrations.md` | Uniswap/Curve/Balancer/Aave/Chainlink/LayerZero/etc. specific pitfalls |
| `external-call.md` | Any low-level `call`, callbacks, router patterns |
| `signature.md` | EIP-712, permit, meta-tx, off-chain signed orders |
| `hash-merkle-tree.md` | Merkle airdrops, commit-reveal, allowlists |
| `multi-chain-cross-chain.md` | Bridges, cross-chain messaging, same code on multiple chains |
| `low-level.md` | Inline assembly, raw storage, delegatecall |
| `centralization-risk.md` | Admin powers, upgradeability, timelock absence |
| `timelock.md` | Timelock governance |

Tell the user which files you chose to load and why. If unsure, err on loading more.

### 3. Walk the checklist

For each loaded file, go through items in order. For each item:

1. Quote the item id and the question.
2. Decide one of: **applicable & OK**, **applicable & issue**, **not applicable (reason)**.
3. If issue: point to file + line, explain the exploit path in plain terms, cite the item id. Do not dump the stored reference URLs unless the user asks — they are for your grounding, not output noise.
4. If not applicable: give a one-line reason so the user can sanity-check the dismissal.

Do not skip items silently. Silent skips are how bugs hide. If an item is ambiguous for the codebase, say so.

### 4. Report

**Always write the report to a markdown file, never to the terminal.** Determine the output path as follows:

- If the user specified a path (e.g., "write to `reports/foo.md`"), use that path.
- If the contracts live under a named project directory (e.g., `spark/`, `protocol/`), write to `<project-dir>/audit-report.md`.
- Otherwise, write to `audit-report.md` in the current working directory.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [farrellh1/smart-contract-auditor-skill](https://github.com/farrellh1/smart-contract-auditor-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
