---
trigger: always_on
description: Fact-verification layer v3 — catches hallucinations using 10 verification tiers, self-consistency sampling, isolated-context verification (CoVe), package existence checking (DepScope), and multi-judge arbitration. Cross-checks claims against local files, stored knowledge, live docs, web search, multi-model cross-check, and external fact-checkers. Every contradiction is persisted so the same lie is never repeated. MANDATORY TRIGGERS: 'verify this', 'truth-check this', 'fact-check this', 'shield t
---



# Truth Shield v3 — Hallucination Defence Layer

Claude generates plausible text, not verified truth. Truth Shield checks every factual claim against real sources before presenting it.

v3 adds research-backed upgrades: self-consistency sampling detects uncertain claims before verification, isolated-context verification breaks confirmation bias, DepScope catches hallucinated packages, and multi-judge arbitration resolves conflicts without single-model bias.

---


## Three modes

### Mode 1: Verify After
User says "verify this" after Claude responds. Truth Shield retroactively checks every claim.

### Mode 2: Shield On (passive)
User says "shield on". Every subsequent response is verified before the user sees it. Slower but catches errors before they matter.

### Mode 3: Spot Check
User says "are you sure about X?" — checks that one claim only.


---


## The verification tiers (0–9)

Claims are checked in this order. Each tier is independent — if unavailable, skip it. Early tiers are fast; later tiers catch more.

```
Tier 0:   fact-mcp cache         — instant (previously verified claims)
Tier 1:   Total Recall            — stored knowledge, past corrections
Tier 2:   Knowledge Graph         — code structure (symbols, call chains)
Tier 3:   Local files             — Grep/Read/Glob (ground truth for code)
Tier 3.5: DepScope                — package existence across 19 ecosystems
Tier 4:   Context7                — live library/API documentation
Tier 5:   Graphiti                — relationship memory (entity facts)
Tier 6:   WebSearch               — general knowledge, current events
Tier 7:   Multi-model cross-check — self-consistency + different model family
Tier 8:   MiniCheck               — external fact-checking model
Tier 9:   Multi-Judge Council     — FACTS-style conflict resolution
```


---


## Step 1: Claim extraction

Parse the response and extract every factual claim — statements that are either true or false.

**Hedged statements are not claims.** "I think", "probably", "might" — these signal uncertainty. Only extract statements presented as definite facts.

Categorise each claim:

| Category | Example | Best tiers |
|---|---|---|
| **Code symbol** | "Function X exists at line Y" | 2 → 3 |
| **Code structure** | "Function X calls function Y" | 2 (cypher) |
| **Package/library name** | "Install lodash-utils" | 3.5 (package check) → 4 |
| **API/library behaviour** | "useEffect runs after render" | 4 → 6 |
| **Past decision** | "We chose JWT over sessions" | 1 → 5 |
| **Entity relationship** | "Service A depends on Service B" | 5 → 2 |
| **General knowledge** | "Python was created in 1991" | 6 → 7 |
| **Current state** | "Server runs on port 3001" | 3 (Read config) |

The "Best tiers" column shows where to START — skip tiers that are obviously irrelevant (e.g., don't check Knowledge Graph for a general knowledge claim). Always fall through to subsequent tiers if the best tier returns UNVERIFIED.


## Step 2: Self-consistency pre-screen (v3)

Before full verification, flag claims that Claude itself is uncertain about. This catches hallucinations that sound confident but aren't stable.

```
For each high-risk claim (package names, version numbers, API signatures, dates):

1. Mentally re-derive the claim from scratch — would you give the same answer
   if asked independently, with no memory of what you just said?

2. Rate your genuine internal confidence (not the confidence you displayed):
   - HIGH: would bet on it, have seen it many times
   - MEDIUM: fairly sure but could be wrong
   - LOW: guessing, filling in from pattern-matching

3. Any claim rated LOW → auto-flag as UNCERTAIN, prioritize for verification
   Any claim rated MEDIUM → verify with extra scrutiny (check 2+ tiers)
   HIGH claims → normal verification pipeline
```

This is NOT a verification source — it's a triage step. Claude's self-assessment is unreliable, which is precisely why uncertain claims get escalated to real sources. The value is catching the claims Claude knows (at some level) it's guessing about.


## Step 3: Verification pipeline

For each claim, work through tiers in order until you get a verdict:

- **VERIFIED or CONTRADICTED** → stop checking this claim (verdict found)
- **UNVERIFIED** → continue to next tier (no evidence yet)
- **CONFLICTED** → stop, escalate to Tier 9

**Exception — Tier 8 (MiniCheck):** If available, run MiniCheck as a second opinion on VERIFIED claims from tiers 4-6 (library docs, web search). If MiniCheck disagrees, escalate to Tier 9. Skip MiniCheck for claims verified by local files (Tier 3) or stored knowledge (Tier 1) — those are ground truth.

**If all tiers exhausted with no verdict** → UNVERIFIED (not UNCERTAIN — self-consistency flags are triage hints, not final verdicts).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BAS-More/truth-shield](https://github.com/BAS-More/truth-shield) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
