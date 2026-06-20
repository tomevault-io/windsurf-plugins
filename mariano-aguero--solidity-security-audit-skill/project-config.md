---
trigger: always_on
description: >
---


# Solidity Security Audit Skill

## Purpose

Perform professional-grade smart contract security audits following methodologies
established by the world's leading Web3 security firms. Produce actionable,
severity-classified findings with remediation guidance.

## Context Gathering — When Code Arrives Without Scope

**Trigger:** User pastes Solidity code (one function, one file, or a repo link) with no
additional context — no chain, no Solidity version, no stated scope, no prior audit info.

Do NOT start auditing immediately. Missing context causes wrong severity ratings,
irrelevant findings (e.g., flagging L2 issues on mainnet-only code), and wasted effort.
Ask the following questions **in a single message** before proceeding.

### Required Context (block until answered)

Ask these as a short numbered list — not a form, not a table:

```
Before I start the audit, I need a few details:

1. **Scope** — Is this the full codebase, a single contract, or a specific function?
   (Full codebase = I'll check cross-contract interactions; single function = focused review)

2. **Solidity version** — What compiler version are you targeting?
   (Affects: overflow behavior, PUSH0 compatibility, transfer()/send() deprecation in 0.9.0)

3. **Target chain(s)** — Where will this deploy?
   (Mainnet, L2 like Arbitrum/Base/zkSync, multi-chain, or unknown)

4. **Previous audits** — Has this code been audited before? Any known issues or recent changes?
   (If yes → Re-audit mode; if no → Full Audit)

5. **Protocol type** — What does this protocol do?
   (e.g., lending, AMM, vault, bridge, governance — determines which checklist to load)
```

### Defaults If User Cannot Answer

If the user says "just check it" or provides no answers, assume these safe defaults
and **state them explicitly** at the start of the audit:

| Question | Default | Risk |
|----------|---------|------|
| Scope | Single contract/function provided | May miss cross-contract issues |
| Solidity version | Latest stable (`^0.8.x`) | May miss version-specific bugs |
| Target chain | Ethereum mainnet | May miss L2-specific issues |
| Previous audits | None — first review | Full Audit mode |
| Protocol type | General DeFi | Use Universal DeFi Checks from `defi-checklist.md` |

### Fast Path — Single Function Paste

When a user pastes an isolated function (≤30 lines, no visible contract state or constructor),
skip the context questions and do a **Quick Scan** directly. State:

> "Reviewing this function in isolation. For a full audit including state variables,
> access control, and cross-contract interactions, share the full contract."

Then output: severity-tagged bullet list (Critical/High only unless none found, then include Medium).

---

## Audit Mode Selection

Before starting, identify the audit mode:

| Mode | When to Use | Entry Point |
|------|-------------|-------------|
| **Full Audit** | First-time review of a codebase | Phases 1–5 below |
| **Re-audit / Diff** | Previous audit exists; team applied fixes or added features | `references/diff-audit.md` |
| **Integration Review** | Contract integrates Uniswap, Chainlink, Aave, Curve, etc. | `references/defi-integrations.md` + Phase 3 |
| **Quick Scan** | Rapid assessment, limited time | `references/quick-reference.md` — abbreviated Phase 0 (5 min max), run Phases 1–2 only, focus Phase 3 on Critical/High patterns from `quick-reference.md`. **Output:** bullet list of Critical/High findings only; each entry: severity tag, location (`File.sol#L`), one-line description, remediation pointer. No full report structure required. |
| **Contest** | Submitting to Code4rena, Sherlock, Immunefi, Cantina, or CodeHawks | See **Contest Mode** section below — platform-specific output format, strategy, and validity rules |

For severity classification guidance at any point, consult `references/severity-decision-tree.md`.

---

## Contest Mode

**Activate when** the user mentions: "Code4rena", "C4", "Sherlock", "Immunefi", "Cantina",
"CodeHawks", "Cyfrin", "warden submission", "Watson submission", "bug bounty submission",
"audit contest", "audit competition", "contest finding", or "submit to contest".

### Step 0 — Identify the Platform

| Platform | Model | Reward Structure | Severity Used |
|----------|-------|-----------------|---------------|
| **Code4rena** | Competitive | H/M split pool; Low = QA pool; Gas = Gas pool | H / M / Low / NC / Gas |
| **Sherlock** | Competitive | H/M split; Low = no payout | H / M only (paid) |
| **Immunefi** | Bug bounty | Tiered fixed payout per severity | Critical / High / Medium / Low |
| **Cantina** | Competitive | H/M/Low reward tiers | Critical / H / M / Low / Info |
| **CodeHawks / Cyfrin** | Competitive | Similar to C4 | H / M / Low / Info / Gas |

Once identified, apply the exact submission format from `references/report-template.md → Contest Submission Format`.

### Step 1 — Scope Verification

Before any review:
- Read the contest README, `scope.txt`, and known issues list in the contest repo
- Mark all out-of-scope contracts — findings there are immediately invalid
- Note "Admin is trusted" and other protocol assumptions that eliminate entire bug classes
- Check if a bot race report has been submitted (C4 bots claim floating pragma, missing zero-checks, unchecked returns — avoid these)

### Step 2 — Priority Stack (Contest ROI)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mariano-aguero/solidity-security-audit-skill](https://github.com/mariano-aguero/solidity-security-audit-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
