---
trigger: always_on
description: Academic English polishing and AI-pattern removal for scientific manuscripts. Default combines SCI-level polish + de-AI detection + rewrite. Supports -p (polish only), -d (de-AI only), -t (top-conference polish standard for NeurIPS/ICLR/ICML). Slash command is /fi (user-configurable). Use when user mentions "polish English", "de-AI", "reduce AI detection", "remove AI traces", or wants to improve English academic text for SCI/CS conferences.
---


# FreeIndulgence: Academic Polish & AI Pattern Removal

## Quick Start

```
/fi                                 # Default: SCI polish + de-AI
/fi -dp                             # Same as default (explicit)
/fi -p                              # SCI polish only
/fi -p -t                           # Top-conference polish only
/fi -t                              # Top-conference polish + de-AI
/fi -d                              # De-AI only
/fi path/to/file.tex                # Process a file with default mode

INVALID: /fi -d -t                  # -t requires polish, but -d skips polish
```

> **Note**: The slash command `/fi` can be renamed by editing the `name` field in this file's frontmatter or by the user's OpenCode/Claude Code configuration. The command name is not hardcoded.

---

## Mode Branching Logic

Before starting the workflow, determine which steps are active:

| Flag | Active Steps |
|------|-------------|
| (none) or `-dp` | Steps 1→2→3(SCI)→4→5→6→7→8→9→10→11 (full pipeline) |
| `-p` | Steps 1→2→3(SCI)→11 (polish only) |
| `-p -t` | Steps 1→2→3(TOP)→11 (top-conference polish only) |
| `-t` | Steps 1→2→3(TOP)→4→5→6→7→8→9→10→11 (top polish + full de-AI) |
| `-d` | Steps 1→2→4→5→6→7→8→9→10→11 (de-AI only, skip polish) |
| `-d -t` | **REJECT** — explain: "-t upgrades polish, but -d skips polish. Use -t without -d for top-conference polish + de-AI." |

---

## How AI Detectors Work — and Our Counter-Strategy

AI detectors (Turnitin, GPTZero, Originality.ai) don't look for specific words. They measure two statistical signals:

### Perplexity — Word Predictability

Given the preceding text, how predictable is the next word?

- **High perplexity** = unpredictable, varied word choices → reads as **human**
- **Low perplexity** = each word is the statistically most likely choice → reads as **AI**

GPT models produce low-perplexity text: the vocabulary is standardized, every word is the "obvious" choice. Claude naturally has higher perplexity — richer vocabulary, less predictable phrasing.

**Our counter-strategy**: After rewriting, sanity-check: if every word in a sentence feels like the most predictable choice, deliberately vary 2-3 words to equally valid but less common alternatives. This isn't about using obscure words — it's about breaking the statistical fingerprint of "the model always picks the median word."

### Burstiness — Sentence Length Variation

How much does sentence structure vary within a paragraph?

- **High burstiness** = sentences of dramatically different lengths, mixed structures → reads as **human**
- **Low burstiness** = sentences all similar length, uniform structure → reads as **AI**

GPT produces low-burstiness text: structured parallel sentences, uniform length. Claude naturally has higher burstiness — long sentences mixed with fragments, colloquial shifts, complex modifiers.

**Our counter-strategy**: After every 2-3 sentences of similar length, deliberately break the pattern. Insert a 3-word sentence. Merge two medium sentences into a long complex one. Use a sentence fragment for emphasis. The rhythm should feel like a person thinking, not a machine formatting.

### The Combined Effect

A detector's confidence score comes from the **interaction** of these two signals. Low perplexity + low burstiness = near-certain AI. High perplexity alone isn't enough if sentence structure is uniform. High burstiness alone isn't enough if every word is predictable.

**Our rewrite must address both simultaneously**: varied vocabulary (perplexity) AND varied sentence architecture (burstiness).

---

## Content Segmentation (MANDATORY first step)

Before any processing, segment the input into content zones. Different zones have different rules:

### Zone Types

| Zone | Examples | Polish Rule | De-AI Rule |
|------|----------|-------------|------------|
| **Prose** | Paragraphs, sentences, captions | Full polish | Full de-AI |
| **Code Block** | Fenced ``` blocks, `\begin{lstlisting}`, `\begin{algorithm}`, `\begin{verbatim}`, `\begin{minted}`, indented code (4 spaces) | **SKIP entirely** | **Skip code body**, scan **comments** (auto-rewrite on confirm) and **print/log strings** (report only) |
| **String Literal in Code** | `print(f"...")`, `logger.info("...")`, `console.log(...)`, `System.out.println(...)`, `raise ValueError("...")`, `assert ... , "..."` | **SKIP** | **Report only** — flag AI patterns in string content, suggest rewrite. Do NOT modify unless user explicitly requests print string modification |
| **Inline Code** | Backtick-wrapped `` `code` ``, `\texttt{...}`, `\lstinline{...}` | **SKIP** | **SKIP** |
| **Math** | `$...$`, `$$...$$`, `\begin{equation}` | **SKIP** | **SKIP** |
| **LaTeX Commands** | `\cite{}`, `\ref{}`, `\label{}`, `\textbf{}` | **PRESERVE** | **PRESERVE** |

### Code Block Comment Handling (Three-Tier System)

**Tier 1 — KEEP (Concise "Why" comments)**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AsdfAlex-learning/free-indulgence](https://github.com/AsdfAlex-learning/free-indulgence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
