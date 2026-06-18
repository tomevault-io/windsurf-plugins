---
trigger: always_on
description: Reference document for the gtm-research slash commands (/competitor-brief, /value-prop, /icp, /discovery, /gtm-research). Loaded by those commands as context. Not for auto-activation — invoke a slash command to start a research run.
---


# gtm-research

A skill for go-to-market research. Given a target company (yours, a prospect's, or a competitor's), produce structured briefs across four dimensions:

1. **Competitor landscape** — who competes, advantages vs disadvantages
2. **Value proposition** — where this company is positioned to win
3. **Ideal Customer Profile (ICP)** — who they should sell to
4. **Discovery questions** — what to ask in a sales conversation

This skill teaches the **research technique**, not the synthesis depth. You'll get a working brief from public web data. You will not get analyst-grade depth — that takes domain expertise and proprietary frameworks.

## How to invoke

Run one of the gtm-research slash commands:

- `/competitor-brief [target]` — competitor advantages vs disadvantages, threat level
- `/value-prop [target]` — where-to-win matrix
- `/icp [target]` — firmographics + personas + anti-ICP
- `/discovery [target] [persona]` — discovery question bank
- `/gtm-research [target]` — runs all four end-to-end

This document is loaded as reference by those commands. It is not auto-activated by natural-language prompts — explicit slash-command invocation only.

## Core technique: parallel multi-source research

The whole skill rests on one principle: **never research with a single tool call when you can fan out**. Single-source research produces shallow, biased briefs. Multi-source fanout + dedupe + synthesis produces briefs worth reading.

### Fanout protocol

For every research turn, issue **multiple tool calls in parallel** before synthesizing. Minimum:

- 1 broad search (the company itself + competitors named in marketing)
- 1 narrow search (the specific dimension you're researching — e.g., "[company] pricing comparison")
- 1+ scrape (the highest-signal URL from the first two)

If your tool host has multiple research APIs (see *Tool host adaptation* below), use all of them in the same turn. Different APIs surface different sources — they are complementary, not redundant.

### Tool host adaptation

This skill works in any agent. Use whatever's available, in this order of preference:

| Capability | Floor (always available) | Ceiling (if configured) |
|---|---|---|
| Search | WebSearch | Exa search + Exa websets, Parallel Search API |
| Scrape | WebFetch | Firecrawl, Stagehand, exa_get_contents (batch) |
| Synthesis | The agent itself | Parallel Task API (pro-fast processor) |

The skill produces useful briefs at the **floor**. The ceiling is faster and deeper, not different in kind. Don't gate on having API keys — start with WebSearch and degrade gracefully.

### Dedupe before synthesis

After fanout, you'll have 10–50 candidate URLs. Before synthesizing:

1. Canonicalize URLs (strip UTM, trailing slashes, fragments)
2. Drop near-duplicates (same domain + path, different query strings)
3. Drop low-signal sources (listicles, AI-generated SEO pages, undated content > 3 years old when freshness matters)
4. Cap to top-N per source category (e.g., max 3 reviews, max 5 product pages)

A clean source set of 8–15 URLs produces better synthesis than a noisy set of 40.

## Per-task playbooks

Each task has a dedicated prompt template in `prompts/`. Use them. They encode the right fanout shape and the right output discipline.

### 1. Competitor research

Goal: produce a list of competitors with **advantages vs disadvantages** for each, grounded in citable sources.

Fanout pattern:
- Search: `"[target] vs"` (G2, Capterra, comparison pages)
- Search: `"[target] alternatives"`
- Search: `"top [category] tools 2026"` (or current year)
- Scrape: target's own marketing site
- Scrape: top 3–5 competitor sites
- Optional: review sites (G2, Capterra, TrustRadius, Reddit)

Synthesize using `prompts/competitor-research.md`. Output: 3–7 competitors with structured advantages/disadvantages.

### 2. Value proposition (where to win)

Goal: identify segments, use cases, and buyer types where the target is positioned to win — and where they should not bother.

Fanout pattern:
- Scrape: target's homepage, pricing page, customers page
- Search: `"[target] case study"` and `"[target] customer"`
- Search: target on review sites (filter for 4–5 star reviews — what works)
- Search: target on review sites (filter for 1–2 star reviews — what doesn't)

Synthesize using `prompts/value-prop.md`. Output: where-to-win matrix (segments × use cases × confidence).

### 3. ICP (Ideal Customer Profile)

Goal: define firmographic + persona criteria for the target's best-fit customers.

Fanout pattern:
- Scrape: target's customer logos, case studies, customer-quotes pages
- Search: `"[target] customers"`, `"[target] case study"`
- Cross-reference customer companies on LinkedIn / Crunchbase (industry, size, geography)

Synthesize using `prompts/icp.md`. Output: firmographics (industry, size, geo, tech stack) + personas (titles, pain, triggers).

### 4. Discovery questions

Goal: produce a discovery question bank for sales calls with prospects matching the ICP.

Fanout pattern:
- Reuse competitor + value-prop + ICP outputs as context

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [salesgraph-inc/gtm-research-skills](https://github.com/salesgraph-inc/gtm-research-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
