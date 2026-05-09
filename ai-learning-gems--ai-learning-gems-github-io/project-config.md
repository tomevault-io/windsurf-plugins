---
trigger: always_on
description: Zero World Knowledge principle, source verification, training data boundaries, sub-agent rules, observed failure patterns
---


# Source Integrity Rules

Shared source verification and world-knowledge rules for all textbook chapter workflows (write, update, edit). These rules exist because LLMs confidently produce plausible-sounding but incorrect information about well-known topics. Every rule below was motivated by a real failure in a real chapter-writing session.

---

## Why Source Integrity Matters

A textbook chapter that gets a quote slightly wrong, rounds a statistic, or describes a framework from memory instead of from its creator's actual words has *silently corrupted the reader's knowledge*. The reader trusts the textbook. They will cite the wrong number in their own paper. They will misattribute the quote. They will describe the framework incorrectly to their students. The corruption propagates.

The only way to prevent this is to treat every specific claim as unverified until it is confirmed against a downloaded, readable source file.

---

## Zero World Knowledge Principle

**You know nothing about the topic except what you read from the downloaded sources.**

You are a skilled writer and organizer, but you have **zero reliable knowledge** about the chapter's topic. Your training data may contain information about the topic, but that information may be outdated, incomplete, or wrong. You MUST NOT:

- Quote an author from memory (even a famous, widely-known quote)
- Cite a statistic you "know" without reading the source
- Describe a method, framework, or concept from training data instead of from a downloaded source
- Fill in gaps when a source is unavailable by "remembering" the content
- Assume a well-known fact is correct without verifying it in a source

**If you cannot find a claim in a downloaded, readable source file, the claim does not exist for you.** Drop it, or download a source that contains it.

---

## What Training Data Can and Cannot Be Used For

The Zero World Knowledge Principle is strict, but not absolute. There is a precise boundary.

### Hard Ban (Requires a Downloaded Source)

| Category | Why It Must Be Sourced | Example of Failure |
|---|---|---|
| **Direct quotes** | Exact wording matters. Training data paraphrases, combines, and misattributes. | Attributing "writing is a primary mechanism for doing research" to Peyton Jones when the transcript says something different. |
| **Statistics and numbers** | Specific numbers drift. A "21%" becomes "20%." A sample size gets rounded. | Writing "6.5-16.9% of reviews" when the paper says "15.8%." |
| **Named frameworks and methodologies** | The creator's original formulation may differ from popularizations. | Describing the "ABT framework" without reading Olson's actual writing, getting the structure wrong. |
| **Specific claims about what an author said, argued, or found** | Training data conflates authors, misattributes findings, merges claims from different papers. | Writing "Pinker argues X" when Pinker actually argues something subtly different. |
| **Paper titles, author lists, venues, and years** | Training data frequently gets these wrong, especially for recent papers. | Attributing a paper to "Liang et al." when the actual authors are "Russo Latona et al." |
| **Descriptions of specific papers, blog posts, or talks** | What a specific work contains must come from reading it. | Claiming a paper "found X" without reading it to verify. |

### Acceptable (No Source Required)

| Category | Why It's Okay | Example |
|---|---|---|
| **Pointing to well-known people as examples** | You are citing them as instances of a category, not claiming what they said. | "Karpathy's blog posts are widely read." "Lilian Weng writes survey-style posts." |
| **General domain knowledge** (non-controversial, non-attributed) | Statements any practitioner would agree with. | "NeurIPS, ICML, and ICLR are top AI conferences." "LaTeX is standard for CS papers." |
| **Structural and rhetorical devices** | How you organize the chapter, analogies, narrative framing. | Using a running example. Creating comparison tables. |
| **Common vocabulary and definitions** | Terms whose meaning is standardized. | "An abstract summarizes the paper." "Ablation studies remove components one at a time." |

### The Litmus Test

Before writing any claim, ask: **"Am I making a specific claim that could be wrong?"**

- "Karpathy writes clearly" → general characterization, okay.
- "Karpathy writes in his blog that X" → specific claim, requires a source.
- "The ABT framework stands for And, But, Therefore" → named framework, requires a source.
- "AI conferences have high submission volumes" → general knowledge, okay.
- "Jiang et al. found that writing quality predicts acceptance across 28,000 submissions" → specific statistic, requires the actual paper.

**When in doubt, download the source.** Minutes to download. The reader's trust to lose.

---

## Source Readability Verification

**Downloaded is not the same as readable.** A PDF in the source folder is useless if it has never been extracted to text.

For each source folder, verify:
1. Does it contain at least one `.md`, `.tex`, or `.txt` file with >500 characters?

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AI-Learning-Gems/AI-Learning-Gems.github.io](https://github.com/AI-Learning-Gems/AI-Learning-Gems.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
