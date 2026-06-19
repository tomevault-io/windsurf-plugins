---
trigger: always_on
description: Deep venture capital due diligence analysis using a multi-agent architecture. Use this skill whenever the user wants to analyze, evaluate, research, or diligence a startup, project, protocol, token, or company from an investor perspective. Triggers on phrases like 'analyze this project', 'do DD on', 'VC analysis', 'evaluate this startup', 'investment memo', 'diligence this', 'should I invest in', 'what do you think about [project]', 'analyze these founders', 'competitive analysis of', or any req
---


# VC Analyst Skill

Perform institutional-grade venture capital due diligence using a **multi-agent architecture**. The main agent acts as a thin orchestrator (like a managing partner) while specialized sub-agents do deep research in isolated context windows (like junior analysts), returning structured JSON summaries.

## Why Sub-Agents?

VC due diligence is context-expensive. A single project analysis can require crawling websites, scraping social metrics, fetching on-chain data, reading GitHub repos, and cross-referencing competitor landscapes. Running this in the main context window degrades quality fast — especially for batch analysis of multiple projects. Sub-agents solve this:

- Each sub-agent works in its own fresh context window
- Only structured summaries return to the main agent (~300-500 tokens per sub-agent vs ~15,000+ tokens of raw web content)
- Multiple projects can be analyzed in parallel
- Multiple dimensions of the SAME project are analyzed in parallel
- If one source blocks or errors, it doesn't pollute the main context

## Architecture Overview

```
Main Agent (Managing Partner / Orchestrator)
│
├─ Phase 1: Input Parsing (inline, lightweight)
│   └─ Parse project list → structured entries
│   └─ Identify: name, URL, socials, founder names, chain/sector
│
├─ Phase 2: Per-Project Dispatch (parallel per project)
│   │
│   │  ┌─ Project Alpha ──────────────────────────────────┐
│   │  │                                                   │
│   │  │  PROJECT COMPONENT (parallel sub-agents):         │
│   │  │  ├─ Product Scout ─────── product, roadmap,       │
│   │  │  │                        philosophy, UX, docs    │
│   │  │  ├─ Competitive Intel ─── moats, competitors,     │
│   │  │  │                        differentiation         │
│   │  │  ├─ Traction Analyzer ─── social metrics,         │
│   │  │  │                        on-chain data, revenue  │
│   │  │  └─ Tech Deep Dive ────── GitHub, tech stack,     │
│   │  │                           code quality, activity  │
│   │  │                                                   │
│   │  │  FOUNDER COMPONENT (parallel sub-agents):         │
│   │  │  ├─ Background Scout ──── education, career,      │
│   │  │  │                        credentials, fit        │
│   │  │  ├─ Social Influence ──── Twitter, LinkedIn,      │
│   │  │  │   Scanner              audience quality,       │
│   │  │  │                        thought leadership      │
│   │  │  └─ Track Record & ────── past projects, exits,   │
│   │  │     Wealth Analyzer       funding raised,         │
│   │  │                           wealth signals          │
│   │  └───────────────────────────────────────────────────┘
│   │
│   │  ┌─ Project Beta (same structure, parallel) ─────────┐
│   ├──│  ...                                               │
│   │  └────────────────────────────────────────────────────┘
│   └─ ... (one pipeline per project)
│
├─ Phase 3: Cross-Project Comparison (inline, if multiple projects)
│   └─ Relative strengths, positioning matrix, ranking
│
└─ Phase 4: Investment Memo Synthesis (inline)
    └─ Compile structured memo with scores and recommendation
```

## ZERO HALLUCINATION POLICY

**This is the single most important rule in this skill. Violating it destroys the entire value of the analysis.**

Every sub-agent and the orchestrator MUST follow these rules:

1. **If you didn't find it, say "Not found" or "Insufficient data."** Never infer, guess, or fill in plausible-sounding details. A blank field with "Not found — searched [sources]" is infinitely more valuable than a fabricated answer.

2. **Distinguish between CONFIRMED, ESTIMATED, and NOT FOUND:**
   - ✅ **Confirmed** — you found this specific data point from a named, citable source
   - 📐 **Estimated** — you derived this from indirect evidence (state your reasoning and evidence)
   - ❌ **Not found** — you searched and came up empty (list what you searched)
   
3. **Every claim needs a source.** If a sub-agent returns a metric or fact, it MUST include where it came from. "Revenue: $5M ARR" is useless. "Revenue: ~$5M ARR (📐 estimated from 500 enterprise customers × ~$10k/yr pricing page)" is useful. "Revenue: Not found — checked Crunchbase, press releases, Token Terminal, pricing page" is also useful.

4. **Confidence tagging is mandatory.** Every sub-agent output includes a `confidence` field. This is NOT optional decoration — it directly informs how the orchestrator weighs the data in the final memo:
   - **High** — multiple corroborating sources, recent data, official sources
   - **Medium** — single source, or data >6 months old, or estimated from indirect evidence
   - **Low** — sparse data, only social media mentions, or significant gaps in what you'd expect to find


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yoitsyoung/vc-analyst-skill](https://github.com/yoitsyoung/vc-analyst-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
