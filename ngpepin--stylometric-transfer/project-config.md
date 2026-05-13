---
trigger: always_on
description: > Guidance for automated coding agents working on `stylometric-transfer`
---

# AGENTS.md

> Guidance for automated coding agents working on `stylometric-transfer`

This document describes the **purpose, architecture, conventions, constraints, and development roadmap** for the `stylometric-transfer` project so that an automated agent can reliably continue development without prior conversation context.

---

## 1. Project Mission

`stylometric-transfer` implements:

- **Stylometric profiling** — extracting an explicit, interpretable style model from an author’s writing corpus
- **Author-conditioned style transfer** — rewriting text to match that style while preserving meaning

The defining design principle is:

> **Explicit, interpretable, versionable style models** rather than fine‑tuning or opaque embeddings.

The system must:
- Remain inspectable by humans
- Keep the style model as JSON
- Avoid fine‑tuning or hidden representations
- Preserve meaning strictly in rewriting

---

## 2. Conceptual Terminology (Canonical)

Use the following terms consistently in code, docs, and comments:

- **Stylometry** — quantitative analysis of writing style
- **Stylometric profile / style fingerprint** — the JSON artifact
- **Style transfer** — rewriting while preserving semantics
- **Author‑conditioned generation** — generation guided by fingerprint
- **Explicit style model** — rule‑ and feature‑based JSON

Avoid ambiguous terms like “clone” in public documentation.

---

## 3. Current Architecture

### Files

- `fingerprint_style.py`
  - Input: compressed corpus archive (`.zip`, `.tar*`)
  - Output: `style_fingerprint.json`
  - Responsibilities:
    - Extract archive
    - Read text files
    - Compute stylometric measurements locally
    - Filter out blockquotes, reference sections, footnotes, inline citation markers, and boilerplate notices (copyright/terms/privacy) from style analysis
    - Detect fiction vs non-fiction; in non-fiction, multi-word quotations are excluded from profiling (override with `--fiction` / `--non-fiction`)
    - Strip embedded BASE64 images from prompts
    - Select representative excerpts
    - Call LLM to synthesise fingerprint JSON
    - If chunking excerpts, merge partial fingerprints via a dedicated LLM merge prompt
    - Prefilter likely proper-name phrases (honorifics + capitalization-ratio heuristics) and rank phrase candidates with the LLM to drop proper names
    - Optionally rank rare-word candidates with the LLM (shared with common-phrase validation) to de-prioritize proper names
    - Validate common phrases via a separate LLM pass to remove OCR/citation noise (with deterministic prefilters for proper names, entity blacklist matches, and date patterns; disable with `--no-phrase-validation`)
    - Repair invalid JSON if necessary
    - Normalize verbose/duplicative `controls.rewrite_policy` clauses and filter `priority_order` to short tokens before writing the fingerprint
    - Normalize lexicon spelling to a US baseline (except literal hard avoids)
  - CLI short flags: `-c` (config, optional; defaults to `./config.llm.json` if present, else next to script), `-a` (archive), `-o` (out), `-v` (verbose)
  - Extra: `--max-prompt-tokens` overrides chunking threshold
  - Defaults: if `--profile-id` or `--author-name` are omitted, both default to the output filename without the `.json` extension

- `apply_fingerprint.py`
  - Input: fingerprint JSON + Markdown file
  - Output: rewritten Markdown + deviations report
  - Responsibilities:
    - Measure input text
    - Detect fiction vs non-fiction; in non-fiction, multi-word quotations are preserved verbatim (override with `--fiction` / `--non-fiction`)
    - Preserve blockquotes, reference sections, footnotes, and inline citations verbatim (excluded from style transfer)
    - Strip embedded BASE64 images before prompt and re-insert after rewrite
    - Call LLM with fingerprint + measurements
    - Enforce preservation of meaning
    - Score style compliance locally and retry with delta feedback (disable with `--no-style-retry`)
    - Apply `general-guidelines.md` humanizer rules when available, using an LLM parser by default then deterministically filtering out conflicts (disable with `--no-humanizer-llm-parse` or `--no-humanizer-guidelines`)
    - Cache parsed humanizer rules in `humanizer_rules.cache.json` (script directory) and re-parse only when guidelines change
    - Sanitize trailing parenthetical/comma qualifiers in headings when enabled (humanizer_mandatory)
    - Apply heading-case normalization according to `humanizer_mandatory.heading_case_normalization`:
      - `automatic` (no deterministic heading-case transform)
      - `identical` (restore source heading case)
      - `by-level` (per-level policy via `heading_case_by_level`)
    - Optionally preserve source proper-name casing in deterministic heading transforms (`preserve_proper_name_case`)
    - Normalize verbose/duplicative `controls.rewrite_policy` clauses and filter `priority_order` when loading a fingerprint
    - Normalize lexical avoidance checks to US spelling for matching, then apply local spelling to final output
    - Return rewritten text and deviations
    - In verbose mode, report per-chunk attempt scores and when best-attempt selection overrides the last attempt

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ngpepin/stylometric-transfer](https://github.com/ngpepin/stylometric-transfer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
