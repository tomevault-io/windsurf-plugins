---
trigger: always_on
description: Vision-enabled verification gate with web search. Use when users want to (1) verify slides/reports/PDFs/images against authoritative online sources, (2) validate that citations actually exist and say what's claimed, (3) check charts/graphs/tables for accuracy, (4) audit AI-generated content in doc-only mode (no external knowledge). Two modes - search mode validates against web, doc-only mode ensures everything traces to provided documents. Supports content in any language.
---


# Citation & Hallucination Checker v2

Verification tool with vision + web search. Validates every claim against authoritative sources or provided documents. Works with content in any language.

**Design principle:** Deterministic verification. Same input → Same output.

---

## Two Verification Modes

### Mode 1: Search Verification (Default)

- Searches web for authoritative sources
- Validates citations actually exist
- Checks if cited sources say what's claimed
- Finds original data for statistics

### Mode 2: Doc-Only Verification

- User provides source document(s)
- EVERYTHING must trace to those docs
- Flags anything that appears to come from external knowledge
- Trigger: "only use this document" / "verify against the PDF only" / "don't search the web"

---

## Two-Pass Architecture

**Critical:** Always use two separate passes. Never interleave extraction and verification.

### Pass 1: Extraction Only

1. Read entire document/slides/images
2. Extract ALL claims using the Claim Extraction Rules below
3. Output numbered list: `[claim_id] | [claim_text] | [claim_type] | [location]`
4. **NO verification in this pass**
5. Present extraction to user for confirmation before proceeding

### Pass 2: Verification Only

1. Take Pass 1 output as fixed input
2. Verify each claim_id in sequential order
3. **NO re-extraction allowed** — work only with Pass 1 claims
4. Apply Status Decision Tree to each claim
5. Generate final report

This prevents "discovering new claims" mid-verification and ensures consistency.

---

## Claim Extraction Rules (Exhaustive)

Extract ONLY these claim types. Apply rules strictly — no judgment calls.

### EXTRACT as claims:

| Type            | Pattern                                                    | Example                                          |
| --------------- | ---------------------------------------------------------- | ------------------------------------------------ |
| **Statistic**   | Any number with unit/context (%, $, count, ratio, decimal) | "92.3% accuracy", "$4.7B market"                 |
| **Comparative** | X is [comparative] than Y                                  | "3x faster than baseline"                        |
| **Temporal**    | Time-bound assertion                                       | "In 2024, adoption reached..."                   |
| **Attribution** | Claim tied to source                                       | "According to WHO...", "Smith et al. found..."   |
| **Causal**      | X causes/leads to/results in Y                             | "This reduces latency by..."                     |
| **Existence**   | Asserts something exists/is true                           | "There are 500M users", "The model supports..."  |
| **Ranking**     | Position claims                                            | "largest", "first", "top 3"                      |
| **Quote**       | Direct quotation                                           | Any text in quotation marks attributed to source |

### DO NOT extract as claims:

| Type                              | Example                                            | Reason                          |
| --------------------------------- | -------------------------------------------------- | ------------------------------- |
| Definitions                       | "Machine learning is a subset of AI"               | Definitional, not factual claim |
| Opinions marked as such           | "We believe...", "In our view..."                  | Explicitly subjective           |
| Hypotheticals                     | "If adoption continues...", "Could potentially..." | Speculative                     |
| Questions                         | "What drives growth?"                              | Not an assertion                |
| Future predictions without source | "Will reach $10B by 2030"                          | Unless citing a forecast report |
| Methodology descriptions          | "We used PyTorch 2.0"                              | Process, not factual claim      |
| Acknowledgments                   | "Thanks to our collaborators"                      | Not verifiable                  |

### Extraction Output Format

```
[C01] | "Model achieves 96.555% accuracy on ImageNet" | Statistic | Slide 3, bullet 2
[C02] | "Outperforms GPT-4 by 12% on reasoning tasks" | Comparative | Slide 3, bullet 3
[C03] | "According to Chen et al. (2024), transformers scale linearly" | Attribution | Slide 5, para 1
[C04] | "Market size reached $4.7B in 2024" | Statistic + Temporal | Slide 7, chart title
```

---

## Status Decision Tree

Apply this tree to EVERY claim. Follow exactly — no shortcuts.

```
START
│
├─ Is this a CITATION claim (references a paper/report/source)?
│   ├─ YES → Go to CITATION VALIDATION
│   └─ NO → Go to STATISTIC/FACT VALIDATION
│
│
CITATION VALIDATION
│

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [serenakeyitan/citation-check-skill](https://github.com/serenakeyitan/citation-check-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
