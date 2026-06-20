---
trigger: always_on
description: |
---


# Journal Compass

A skill that studies one journal and builds a standalone `<journal-slug>-fit` companion for it.
The companion supports four steps: **TOPIC → WRITE → POLISH → SUBMIT**.

---

## Inputs

What the user can provide:

| Input | Effect |
|-------|--------|
| Journal name (+ publisher / ISSN) | Required — disambiguates look-alikes |
| PDFs of the journal's papers | Improves writing-framework extraction significantly |
| Author's own draft or abstract | Companion runs a fit check on it as a bonus deliverable |
| Rival journal name | Required for the rival-journal test; pick the closest competitor if the user has none |

---

## Core method

**Three evidence streams:**

| Stream | Where | What it tells you |
|--------|-------|------------------|
| **Claims** | Aims & Scope, editorials, author guidelines | The official position |
| **Published** | Recent papers + open-access full texts | What actually gets accepted |
| **Rejected** | Reviewer guidelines, desk-reject signals, author reports | The hidden filter |

**The claims-vs-reality gap** is the most useful finding. When a journal says it welcomes X but publishes under 5% X, that gap is a desk-reject risk for X-only submissions. Always compare the two streams.

**Three stages every submission must clear:**

```
Stage 1 — Editor's desk     (~30 seconds): scope, contribution type, obvious disqualifiers
Stage 2 — Peer review       (weeks):       rigor, framing, novelty standard
Stage 3 — Accepted paper shape:            abstract recipe, structure, title pattern, word limits
```

The companion maps your work to each stage so you know *which* stage fails, not just whether a paper fits.

**Rival-journal test (noise filter):**

A finding goes into the profile only if it would change which of two similar journals you'd submit to.
If both journals want it equally, it's a field-wide norm — discard it.
This is why you always compare against one rival journal.

---

## Build workflow

Seven steps. Stop at the two ⏸ checkpoints for explicit author confirmation before proceeding.

### Step 1 — Frame the build

Confirm before starting research:

- **Exact journal** — full name + publisher (e.g. *Computers & Education*, Elsevier). Disambiguate look-alikes (*Computers & Education* vs *Computers & Education: Artificial Intelligence*).
- **Rival journal** — one similar journal for the test. If the author has none, pick the closest competitor yourself and name it.
- **Material on hand** — any PDFs of the journal's papers? The author's own draft or abstract? First-hand papers beat anything scraped.
- **Depth** — quick (Stage 1 editor-screen only) or full (all four companion steps, default: full).
- **New or update** — scan `.claude/skills/*-fit/`. If one exists, switch to "Updating an existing skill" at the end of this file.

Create the self-contained evidence folder **before starting research** (so subagents have a place to write):

```
.claude/skills/<journal-slug>-fit/
├── SKILL.md                          # final product (built in Step 6)
└── references/
    ├── evidence/
    │   ├── claims.md                 # what the journal says it wants
    │   ├── published.md              # what it actually publishes (≥5 papers)
    │   ├── rejected.md               # what gets rejected and why
    │   ├── guidelines.md             # author guidelines → submission kit elements
    │   ├── writing-framework.md      # section-by-section moves (from OA full texts)
    │   └── rival-<slug>.md           # comparison against the rival journal
    └── corpus/                       # saved PDFs, guideline snapshots
```

### Step 2 — Gather evidence

Fill each `evidence/*.md`. If subagents are available, run these in parallel (one per stream). Without subagents, run sequentially — the process is identical.

**Every claim must have a source URL and a confidence tag: `[HIGH]` = verbatim from official source; `[MED]` = official Elsevier/Springer/etc. general policy; `[LOW]` = inferred.**

---

#### `claims.md` — what the journal says it wants

Prompt for this stream:

```
Research <journal name> (<publisher>, ISSN <if known>).

Find and record:
1. The full Aims & Scope text — quote it verbatim.
2. The article types it accepts (research paper, review, short communication, etc.) and any explicit exclusions.
3. Any recent editorials or editor statements about scope, direction, or what they want more/less of.
4. The explicit "we do not publish X" statements, if any.

For each item, record: what it says, the source URL, and a confidence tag [HIGH/MED/LOW].
Distinguish between hard rules ("papers must be X") and soft guidance ("we prefer Y").

Write findings to: <skill-folder>/references/evidence/claims.md
```

---

#### `published.md` — what the journal actually publishes

Prompt for this stream:

```
Research what <journal name> has actually published in the last 2–3 years.

Use OpenAlex, Semantic Scholar, or the journal's own website to find recent papers.
If $ARIS_REPO/tools/openalex_fetch.py is available, use it to pull the corpus efficiently.
Analyze ≥5 papers (the more the better).

Record:
1. Topic clusters — what subjects appear most, with rough frequency estimates.
2. Contribution-type mix — approximately what % is: empirical / theory / methods / review / design.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Youn-17/journal-compass](https://github.com/Youn-17/journal-compass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
