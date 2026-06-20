---
trigger: always_on
description: Find academic papers across 5 sources (OpenAlex / Semantic Scholar / CrossRef / PubMed / arXiv) with adjustable depth — Quick scan (5 min) to Audit prep (3 hr). Use when the user wants to find papers, run a literature search, gather references, or scope a research topic. Triggers on: search verbs ('find papers', 'literature search', 'papers about X'), review types ('scoping review', 'systematic review', 'SR prep', 'literature review', 'lit review', 'help me write a lit review'), Chinese ('找文献', 
---


# paper-search-pro

Multi-source literature search with adjustable depth. Four tiers, five data sources orchestrated by you (the main agent). Python helpers handle deterministic work; LLM classification is delegated to parallel Inline SubAgents — no external API key required.

## When to use this skill

- User wants to find academic papers / 找文献 / 论文搜索
- User is preparing a literature review, systematic review (SR), scoping review, or meta-analysis
- User wants to scope research on a topic for a thesis / proposal / coursework / news story
- User asks "what research exists on X" / "find me papers about Y"
- User uploads a query that suggests literature gathering (PICO, SPIDER, MeSH, RCT, etc.)

## When NOT to use

- User wants to **read** a specific paper (use PDF reader / download tool)
- User wants to **summarize** a single known paper (use a summarizer)
- User wants to **download** PDFs given DOIs (use `paper-downloader-portable`)
- User already has a literature set and wants to write a review (use `literature-set-review` / `factor-outcome-review`)
- User wants concept explanation, not papers ("what is prospect theory" → just answer)

---

## 🔥 Execution discipline (read this BEFORE running anything)

These four rules govern every step below. Violating them is the dominant failure mode observed in real sessions — re-read them whenever you feel rushed.

### Rule A — NEVER `cd` into the Skill directory

**Reason**: `cd $PSP_HOME` rebinds `./` to the Skill asset directory. Every `./paper-search-results/...` after that lands inside the Skill folder, not where the user is working. Re-installing the Skill overwrites history; the user can't find outputs in their own working directory.

**Correct pattern** — execute helpers from the user's working directory using `PYTHONPATH`:

```bash
PYTHONPATH=$PSP_HOME \
  python3 -m scripts.openalex_helper search "<query>" --limit 30 \
  > "$SEARCH_DIR/raw/openalex.json"
```

Where `$PSP_HOME` is the Skill install directory (resolved in STEP 0) and `$SEARCH_DIR` is an **absolute path under the user's PWD** (also from STEP 0). Shell `cwd` remains the user's PWD; `./` paths resolve to where the user expects.

### Rule B — Parallelism is MANDATORY for SubAgent dispatch

When you launch classifier SubAgents (STEP 6), you **must** put up to 5 `Task` tool_use blocks **inside one assistant message**. Serial dispatch (one Task per message, waiting for each result) makes Standard tier take ~17 min instead of ~10. See STEP 6 worked example.

### Rule C — Tell the user every time you skip a step

If you deliberately skip any STEP (because tier budget is exhausted, data is empty, or user preference), state it explicitly:
- **What** you skipped (e.g. "STEP 10 L3 enrichment")
- **Why** (tier? data? user choice?)
- **What's lost** (e.g. "no influentialCitationCount, no funder/license fields")
- **How to recover** (e.g. "re-run at `--tier deep` to include this")

Never skip silently. Skipping is fine; surprising the user is not.

### Rule D — Read the cited references/ file BEFORE the step

Each STEP names a `references/<file>.md`. Read it before running the step's commands — the cheatsheets contain edge cases that are not duplicated in SKILL.md. Average must-read coverage across recent sessions was 5/17 — drive it higher.

---

## Architecture at a glance

```
You (main agent) drive the workflow per this SKILL.md.
Python helpers do deterministic work — NO LLM inside, NO external API key.

  L1 OpenAlex (primary)  → deep top-100 multi-strategy
  L2 PubMed (medical)    → MeSH enricher (mostly; Audit-tier can search independently)
  L2 arXiv (CS/preprint) → T-0~T-4 freshness sentinel
  L3 Semantic Scholar    → influentialCitationCount + abstract fallback
  L3 CrossRef            → funder / license / clinical-trial-number

  Classification         → Inline SubAgents (parallel, file-IPC, 5 per message)
  Output                 → HTML (Shadcn) + MD + BibTeX/RIS/CSV + PRISMA-S log
```

---

## The 4 tiers — pick first

| Tier | Wall-clock | Papers | When to pick |
|------|------------|--------|--------------|
| Quick | ~5-8 min | 20-60 | "查一下" / "几篇" / "before tomorrow" / fast scope |
| **Standard** (default) | ~10-17 min | 60-180 | Scope a topic / write background / general lit search |
| Deep | ~30-45 min | 180-400 | "thorough" / writing a review article / 综述写作 |
| Audit | ~2-3 hr | 400-1000+ | "systematic review" / "PRISMA" / "Cochrane" / "meta-analysis" |

📖 **BEFORE picking, read `references/tier_decision.md`.** Tell the user your choice and why. For Audit, show limitations warning + get explicit confirmation before starting.

---

## The recipe


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [O0000-code/paper-search-pro](https://github.com/O0000-code/paper-search-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
