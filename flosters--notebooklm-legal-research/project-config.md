---
trigger: always_on
description: Legal research workflow that combines Claude's orchestration with Google NotebookLM's deep web research and AI analysis capabilities. Creates a dedicated NotebookLM notebook, runs deep multi-query legal research, performs structured IRAC/CRAC/CREAC analysis grounded in the imported sources, generates a downloadable legal memo, and optionally produces additional artifacts (slide deck, podcast, mind map, quiz, etc.). Trigger on /notebooklm-legal-research or when the user asks to research a legal t
---


# NotebookLM Legal Research

Orchestrate a full legal research workflow using NotebookLM for deep source discovery and analysis, with Claude managing scope, gates, and prompt sequencing.

## Prerequisites

Verify authentication before starting:
```bash
notebooklm status   # Should show "Authenticated as: email@..."
```
If not authenticated: `notebooklm login`

---

## Phase 1 — Gate 1: Scope (Claude)

Extract from the query:
- Legal question / topic
- Jurisdiction — ask if not stated or not clearly inferable
- Area of law — infer if possible; confirm only if genuinely ambiguous
- Procedural posture (litigation, transactional, advisory, academic) — infer when possible
- Report language — detect from the query itself (not from the jurisdiction). Record this explicitly. Ask only if the user writes in one language but seems to want the report in another.

Ask only what's missing. One consolidated question. Never ask more than 3 clarifying questions.

**Determine how many deep research queries are needed (3–5).** Default to 3. Add a 4th or 5th only when the issue genuinely requires it — not as a precaution. Use these signals:

- **3 queries** — single legal issue, well-defined jurisdiction, established doctrine (e.g., "Does X clause constitute unfair competition under Argentine law?")
- **4 queries** — two distinct sub-issues, or a fast-moving area where recent developments need a dedicated angle (e.g., regulatory changes, pending legislation)
- **5 queries** — multi-issue problem, multi-jurisdictional comparison, or highly specialized area where primary authorities, doctrine, enforcement practice, comparative law, and academic criticism each warrant separate coverage

For each query slot, define the angle before presenting the plan. Record the list as `RESEARCH_QUERIES` — it will be used verbatim in Phase 3.

Check for pandoc before presenting the plan:
```bash
pandoc --version 2>/dev/null && echo "pandoc: available" || echo "pandoc: not found"
```

Present a research plan and ask for confirmation:
```
Research question: [precise restatement]
Jurisdiction: [confirmed]
Area of law: [identified]
Report language: [detected from query — e.g., English, Spanish]
Research method: NotebookLM deep web research + structured IRAC/CRAC analysis
Output: Legal memo (.html + .docx)           ← if pandoc found
Output: Legal memo (.html only)              ← if pandoc not found (install: brew install pandoc)
NotebookLM notebook: will be kept for future reference

Research queries ([N] total — each runs sequentially, ~5–10 min each, ~[N×5]–[N×10] min total):
  1. [angle — e.g., primary authorities: case law, statutes, regulations]
  2. [angle — e.g., doctrine and academic analysis]
  3. [angle — e.g., specific angle tailored to the topic]
  4. [angle] ← only if N ≥ 4
  5. [angle] ← only if N = 5

⚠ Deep research queries run one at a time. More queries = broader coverage but longer wait.
  To reduce wait time, remove a query from the list above before confirming.
```
Ask: "Confirm to proceed, or adjust scope?"

---

## Phase 2 — Notebook Creation

```bash
NB_ID=$(notebooklm create "Legal Research: [Topic] — [Jurisdiction] — [YYYY-MM-DD]" --json | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('id') or d.get('notebook_id') or d.get('notebook',{}).get('id',''))")
echo "Notebook ID: $NB_ID"
```
Use `$NB_ID` with `-n $NB_ID` explicitly on all subsequent commands (never rely on `notebooklm use` — parallel safety).

---

## Phase 3 — Deep Research

Run the **N queries confirmed in Phase 1** (between 3 and 5) sequentially, using the exact angles recorded as `RESEARCH_QUERIES`. Capture `research status` immediately after each query completes, before starting the next one. This is critical: `research status` only reflects the most recently completed session, so capturing after each query is the only way to collect sources from all runs.

```bash
# Query 1 — [angle from RESEARCH_QUERIES[1]]
notebooklm source add-research "[query 1 string]" --mode deep -n "$NB_ID"
notebooklm research status --json -n "$NB_ID" > /tmp/research_1_$NB_ID.json

# Query 2 — [angle from RESEARCH_QUERIES[2]]
notebooklm source add-research "[query 2 string]" --mode deep -n "$NB_ID"
notebooklm research status --json -n "$NB_ID" > /tmp/research_2_$NB_ID.json

# Query 3 — [angle from RESEARCH_QUERIES[3]]
notebooklm source add-research "[query 3 string]" --mode deep -n "$NB_ID"
notebooklm research status --json -n "$NB_ID" > /tmp/research_3_$NB_ID.json

# Query 4 — [angle from RESEARCH_QUERIES[4]] — only if N ≥ 4
notebooklm source add-research "[query 4 string]" --mode deep -n "$NB_ID"
notebooklm research status --json -n "$NB_ID" > /tmp/research_4_$NB_ID.json

# Query 5 — [angle from RESEARCH_QUERIES[5]] — only if N = 5
notebooklm source add-research "[query 5 string]" --mode deep -n "$NB_ID"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Flosters/notebooklm-legal-research](https://github.com/Flosters/notebooklm-legal-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
