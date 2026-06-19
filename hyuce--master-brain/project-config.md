---
trigger: always_on
description: Use when inferences or evaluations need to be evidence-backed; when prior reasoning felt premature or conclusion-first; when analyzing a document, claim, decision, or argument; when quality control of a prior analysis is needed; or when a judgment must hold up under adversarial scrutiny. Do NOT use for simple lookups, single-step operations, or routine tasks without a judgment component.
---


# master-brain

## Overview

**No inference without evidence. No conclusion without verification.**

Every analysis passes through a mandatory 5-stage reasoning loop. Every stage is processed as a distinct, demarcated unit of work. The loop repeats from Observation if Verification fails.

**Violating the letter of the rules is violating the spirit of the rules.** This principle binds at every stage. No stage may be skipped or waved off as "obvious."

## When to Use

```
Analysis, decision, or evaluation needed?
├─ No  → Don't use this skill
└─ Yes → Can it be answered in a single step?
          ├─ Yes → Don't use this skill
          └─ No  → USE this skill
```

**Use this skill for:**
- Complex multi-step decisions with trade-offs
- Document analysis: claims, findings, evidence, gaps
- Quality control of structured analysis (yours or someone else's)
- Gap analysis, benchmarking, evaluation against a standard
- Any inference, evaluation, or judgment that must be defensible

**Do not use for:**
- Simple informational queries ("where is this file?")
- Single-step operations ("rename this variable")
- Routine code writing or refactoring
- Tasks with no judgment component

## The 5-Stage Loop

```
OBSERVATION → HYPOTHESIS → EVIDENCE → CONCLUSION → VERIFICATION
   ↑________________________________________________|
              (loop back if verification fails)
```

| # | Stage | Question |
|---|-------|----------|
| 1 | Observation | What is present? |
| 2 | Hypothesis | What could it mean? |
| 3 | Evidence | What supports or refutes each candidate? |
| 4 | Conclusion | What is the final judgment? |
| 5 | Verification | Under what conditions is this wrong? |

**Rules that bind at every stage:**
- Run the stages in order. Do not skip.
- Process each stage as a distinct unit. Do not batch.
- If Verification fails, return to Observation with the new evidence. Do not patch the conclusion.

### Stage 1 — Observation

Collect data **without interpretation**. List facts, not opinions.

**Do:**
- For a document: itemize claims, structure, sections, evidence cited, gaps.
- For a decision: list current state, constraints, stakeholders, timeline, prior attempts.
- For a claim: restate it precisely; identify what would need to be true for it to hold.
- For QA: extract structure, numbering, headings, paragraph count.

**Do not:**
- Add commentary, judgment, or "I think" statements.
- Pre-classify findings (that is Hypothesis work).
- Skip items because they seem irrelevant.

### Stage 2 — Hypothesis

Generate candidate explanations for each observation. Force at least two hypotheses per observation — do not stop at the first plausible reading.

**Do:**
- "This could mean X..."
- "A rule could be violated if Y..."
- "Alternatively, Z is also possible..."

**Do not:**
- Settle on one hypothesis without considering alternatives.
- Treat the first hypothesis as the conclusion.
- Skip the stage because "it's obvious."

### Stage 3 — Evidence

Gather concrete evidence. For each hypothesis, list what supports it and what contradicts it.

**Do:**
- Cite the specific rule, document section, or fact.
- Verify cross-references (does the cited section actually exist?).
- Check statistics, dates, names — every concrete claim.
- Match claims against any supplied domain rules.
- **Annotate each load-bearing claim with a confidence level (High / Medium / Low) and a brief source-quality note.** The Evidence output feeds the weakest-link rule in Stage 4; without per-claim confidence, the rule cannot be applied.

**Do not:**
- Accept a hypothesis on its own plausibility.
- Use vague evidence ("generally accepted that...").
- Skip the contradictions.

### Stage 4 — Conclusion

State the conclusion clearly, with a severity/urgency label, a confidence level, and a concrete recommendation.

**Do:**
- For findings: end with a `[Urgency] [Direction]` label in **bold** and a confidence level. Format: *"Therefore, **[Major Gap]** has been identified. **Confidence: High**."* See "Severity / Urgency Guidance" and "Confidence Level" below for the taxonomy.
- For Type 1 (irreversible) decisions, see "Calibrating Rigor" — **Medium-or-High confidence is required** before issuing a recommendation. If confidence is Low, return to Stage 3 to gather more evidence.
- For decisions: state the recommended action, the reason, the alternative, and the cost of not acting.
- For QA: state pass/fail with reason.
- Recommendations should be concrete and actionable. Use the modal form "should" or "must."

**Do not:**
- Issue a conclusion without a `[Urgency] [Direction]` label.
- Issue a conclusion without a confidence level, or one set without applying the weakest-link rule.
- Issue a finding without a recommendation (a finding without a fix is a complaint, not an analysis).
- Hedge with "I think" or "in my opinion."
- Assign a confidence level you cannot justify with evidence.

### Stage 5 — Verification


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyuce/master-brain](https://github.com/hyuce/master-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
