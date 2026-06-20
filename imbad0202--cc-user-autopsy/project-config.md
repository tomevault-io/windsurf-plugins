---
trigger: always_on
description: Produces a deep, honest peer-review of how someone uses Claude Code by statistically analyzing their local session data (~/.claude/usage-data/ and ~/.claude/projects/). Trigger whenever the user asks to review, analyze, audit, or critique their own Claude Code usage, workflow, habits, or skill level — including phrases like "analyze my cc usage", "review my cc sessions", "peer review my cc workflow", "deeper than /insights", or any ask for an honest audit of their AI workflow. Also covers portfo
---


# Claude Code User Autopsy

Produces an honest, evidence-traceable peer-review of a user's Claude Code workflow.

## When NOT to use

The frontmatter above describes when to trigger this skill. One case to skip:

**If the user only wants one narrow statistic** that a single grep / ls / wc can answer (e.g. "how many sessions did I run this week") — just answer directly. This skill is for holistic peer review, not ad-hoc lookups.

## What this skill produces

A self-contained HTML report at `~/.claude/usage-data/cc-user-autopsy.html` (or `-hr.html` for portfolio audience). The V4 layout is story-first, not dashboard-first:

**SELF audit layout** (private diagnostic letter):

1. **Usage snapshot** §01 — activity panel (cache, models, cost, characteristics) + a 4-tile behavior strip (commits / interactive time / Task agent % / MCP %). Replaces the old 8-tile metric grid. Benchmark caveat at the top.
2. **Reading guide** — short paragraph orienting the four-zone story (when / how / where stuck / cost).
3. **Peer review** §02 — Claude-written story in 4 sections plus a "connecting it back" paragraph. Comes BEFORE scoring so the grid reads as an index, not a verdict.
4. **9-dim scoring** §03 — 9 rule-based scores (1-10) across delegation, root-cause debugging, prompt quality, context management, interrupt judgment, tool breadth, writing consistency, time-of-day management, token efficiency.
5. **This week, try this** §04 — 3-5 hand-curated action items derived from peer-review claims.
6. **Case study** §05 — the strongest single session, opened by a metric strip headline (e.g., "451 min · 14 commits · 56 tests · deploy · fully achieved") and three paragraphs (problem / orchestration / shipped).
7. **Pattern mining** §06 — charts for prompt-length × outcome, friction categories, tool usage, weekday × hour heatmap, helpfulness self-rating.
8. **Weekly trends** §07 — growth curve plus 5 weekly detail charts (sessions / tokens / good rate / friction / prompt length).
9. **Evidence library** §08 — claim-indexed (not by 7 tag buckets). Each peer-review claim shows 2-3 sessions that prove it. Empty claim groups are hidden, not apologetically labeled.
10. **Methodology footer** — small-type appendix, not a full section.

**HR / portfolio layout** differs:

- Profile card + activity panel → Public Artifacts → Peer review (250-350-word candidate memo) §02 → Hiring signals (4 dims, not 9) §03 → Case study §04 → Growth curve + outcome donut §05 → Methodology disclosure.
- Evidence library is **hidden entirely**, no #evidence anchor.
- Scoring chrome reads "Hiring signals" (not "Rule-based scores"), shows only D1 delegation, D2 root-cause, D6 tool breadth, D9 token efficiency. No overall average.
- A short self-awareness caveat under the grid acknowledges that more dimensions exist privately. **Does not name the hidden ones.**
- Pattern mining is dropped entirely. Only growth curve + outcome donut survive.
- Shipped-with-Claude shows at most 3 items, all from the public-projects allowlist. Redacted private projects are dropped, not displayed as "Private platform project" filler.

Both audiences get a **benchmark caveat** disclaimer at the top reminding readers this is unbenchmarked individual data.

## Workflow overview

```
Step 0   → ASK which version (self / hr / both) and which locale (en / zh_TW).
           For HR, collect profile + public-projects allowlist + artifacts BEFORE running.
Step 1a  → scripts/scan_transcripts.py     (merges subagent tokens into parent sessions)
Step 1b  → scripts/aggregate.py            (combines transcript-rows + session-meta + facets)
Step 2   → scripts/sample_sessions.py      (picks 15-24 representative sessions)
Step 3   → Claude writes peer-review.{audience}.{locale}.md  (V4 story format)
Step 3.5 → Claude writes try-this-week.{locale}.md (SELF only)
           Claude writes case-study.{self|hr}.{locale}.md (BOTH audiences, two files)
Step 4.5 → (zh_TW only) rewrite the EN peer review natively into zh_TW
Step 4   → scripts/build_html.py with --peer-review --try-this --case-study --audience --locale
Step 5   → open the HTML in browser and tell the user
```

Each script is idempotent. If a step fails, re-run it.

## Step 0 — Ask first, build second

Before running anything, **ask the user two questions in a single prompt**. Never guess from keywords.

> "I can build two versions of this report:
>   **A. Self audit** — honest diagnostic letter for your eyes only. Shows every project name, session ID, and friction detail.
>   **B. HR / portfolio** — public-facing summary for recruiters. Hides private projects, redacts session IDs, and leads with a profile card.
>   **C. Both.**
> Which one(s)?
>
> Output language:
>   **1. English (default)**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Imbad0202/cc-user-autopsy](https://github.com/Imbad0202/cc-user-autopsy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
