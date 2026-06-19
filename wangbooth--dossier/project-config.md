---
trigger: always_on
description: Use when investigating a topic that benefits from a reusable, citation-backed NotebookLM corpus—literature reviews, due diligence, regulatory/legal research, competitive intelligence, long-term personal knowledge queries, or any repeated deep-dive across the same body of material. Not for one-off web search, single-document reads, or code exploration.
---


# Dossier

Open a topic-scoped NotebookLM corpus (the "dossier"), let Claude orchestrate curation and reasoning, let NotebookLM serve citation-backed answers from within the dossier. The dossier persists—future questions on the same topic reuse it.

## When to use

- Reading 5+ papers/reports on one topic
- Pre-IPO / pre-deal due diligence on a company or sector
- Legal, regulatory, or compliance review across document sets
- Competitive analysis across multiple products/companies
- Long-term personal knowledge base queries (Obsidian/Readwise exports, meeting archives)
- Any task where answers must be traceable to source text

## When NOT to use

- One-off questions answerable from general knowledge or single web page — just ask Claude directly
- Codebase exploration or "jump to definition" — your agent's native file tools (Read/Grep/etc.) are stronger
- Real-time / freshly changing information — NotebookLM is static
- Tiny corpora (< 5k tokens) — fitting in Claude's context is simpler
- When speed matters over depth — NotebookLM is ~3× slower than direct chat

## Setup (first run in a machine)

Preflight has 4 steps. Only step 1 is read-only and always safe to auto-run. Steps 2–4 have side effects (disk writes, browser popups, skill directory changes) — **ask the user before each one**.

### Step 1 — Detect (auto, read-only)

```bash
# What CLI form is reachable?
if command -v notebooklm >/dev/null 2>&1; then
  NBLM="notebooklm"
elif python3 -m notebooklm --help >/dev/null 2>&1; then
  NBLM="python3 -m notebooklm"   # pip-installed but scripts dir not on PATH — common on macOS
else
  NBLM=""                        # not installed
fi

# Does the existing session work?
if [ -n "$NBLM" ] && $NBLM auth check 2>&1 | grep -q "Authentication is valid."; then AUTH_OK=1; else AUTH_OK=0; fi
# Note: matches the final verdict line — only appears when every sub-check
# passed. Matching "✓ pass" alone would false-positive when e.g. only
# "Storage exists" passed but SID/token checks failed.
```

### Step 2 — Install (only if `$NBLM` is empty) — **ASK FIRST**

Tell the user: *"I don't see `notebooklm-py` on this machine. May I install it with `pip install 'notebooklm-py[browser]'` plus Playwright Chromium (~150MB)?"*. Wait for an explicit yes, then:

```bash
pip install "notebooklm-py[browser]"
python3 -m playwright install chromium
NBLM="python3 -m notebooklm"
```

If the `notebooklm` binary lands outside `$PATH` (typical on macOS where `pip install` writes to `~/Library/Python/<ver>/bin/`), **don't try to patch PATH yourself** — just report it and continue using the `python3 -m notebooklm` form everywhere. It works identically.

### Step 3 — Authenticate (only if `AUTH_OK=0`) — **ASK FIRST, OPENS BROWSER**

`$NBLM login` opens Chrome for Google OAuth. Tell the user it'll pop a browser window before running.

### Step 4 — Install the low-level `notebooklm` skill (once per machine) — **ASK FIRST**

`$NBLM skill install` writes to `~/.claude/skills/notebooklm/` — it gives Claude direct knowledge of the raw CLI commands, which complements this `dossier` skill. Low risk but still a filesystem change; tell the user, then run.

## Routes — pick the right entry point

**First branch — does a dossier on this topic already exist?** Run `$NBLM list` before anything else.

- **Yes** (same or adjacent topic) → **Route D** (query / top-up / hygiene). Most time on any long-running topic is spent here, not in fresh builds.
- **No** → pick one of A/B/C below based on source-quality requirements. Default: **Route B**.

Always present the choice to the user before spending time.

| Route | When | Setup time | Claude tokens | Durability |
|---|---|---|---|---|
| **A. NotebookLM deep research** | Quick overview, one-shot curiosity | ~5 min | near-zero | ephemeral (dossier stays but seeded by auto-research) |
| **B. Claude-curated sources** ⭐ | Noisy topic, long-term reuse, authoritative-source requirement | 15–30 min | moderate (fetching/filtering) | persistent |
| **C. User-specified sources** | Strict source whitelist (specific authors, databases) | depends on list | moderate | persistent |
| **D. Existing dossier** | Continuing research on a topic that already has a dossier | near-zero | low (mostly Q&A, maybe small top-up) | persistent |

### Route A — NotebookLM auto-research

```bash
$NBLM create "dossier-<topic-slug>"
$NBLM use <notebook-id>
$NBLM source add-research "<search query>" --mode deep
```

NotebookLM's own research agent picks sources, imports them, and indexes. Claude then `ask`s questions. Source quality is opaque — good for exploration, risky for decisions.

### Route B — Claude-curated (recommended default)

**Steps:**

1. Claude searches authoritative sources (e.g. PubMed, Cochrane, ISSN, SEC EDGAR, primary sources for the topic — not random blogs or SEO pages)
2. Claude drafts a **source shortlist** with a one-line justification per source
3. **User confirms or edits the list** — never skip this step

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wangbooth/dossier](https://github.com/wangbooth/dossier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
