---
trigger: always_on
description: This is the **core rules file** — always loaded, kept lean on purpose. The deep per-area
---

# Lexica — CLAUDE.md

## What this file is (read this first)
This is the **core rules file** — always loaded, kept lean on purpose. The deep per-area
reference (schemas, feature histories, gotchas, incident records) lives in `docs/claude/*.md`.
Those files are NOT optional background: they encode past incidents and load-bearing detail.

**MANDATORY ROUTING RULE:** before editing an area, read its routed doc below — the whole
relevant section, not a skim. If a task spans areas, read all routed docs. Never rely on a
summary in this file where a routed doc has the detail.

| Touching… | Read first |
|---|---|
| Any table, join, schema, data invariant, MetaV/TIPNR, lexica_def, structural cards | `docs/claude/data-model.md` |
| Frontend build, Library tab, reading modes, three-zone shell, Notes/accounts, mobile | `docs/claude/frontend.md` |
| Ask-the-corpus, AI cache, TSK xref, summaries, synthesis prompts | `docs/claude/ai.md` |
| Word study tab, lexicon endpoints, English finder | `docs/claude/word-study.md` |
| Deploy, CI, backups, rebuilds, maintenance scripts, SEO pages, rate limits | `docs/claude/ops.md` |
| Visual design detail | `docs/design.md` (doctrine summary below) |

Active work state lives in `TODO.md` + the `HANDOFF_*.md` / `AUDIT_*.md` docs — never here.

## Overview
Lexica is a Flask-based Greek and Hebrew Bible word study app. ABP (Apostolic Bible Polyglot)
interlinear is the primary text; KJV is a fully searchable parallel corpus. The design is
scholarly but accessible — no prior Greek training required.

Stack: Flask (Python) + SQLite · React 18 (UMD) with JSX in `static/src/*.jsx` precompiled by
Babel to `static/app.js` (committed) · deployed on PythonAnywhere ($10 Dev tier) · repo
`lexica-bible/lexica` on GitHub. `bible.db` lives ONLY on PA, never locally, never in git.

## HOW TO TALK TO THE USER — read every session, no exceptions
Plain, concise English — like a colleague, not a textbook. The user is a data-center engineer
(CCNA, Linux, lots of hands-on) — NOT a programmer. Assume infra/CLI/networking fluency (don't
explain basic console steps), but use NO developer jargon. Avoid words like *idempotent,
transaction, schema, query/SELECT, commit, null, boolean, lock/read-lock, upsert, snapshot* —
translate them into plain terms ("running it again just redoes the same work, no harm"; "that
command only reads the database, it never changes it"). Short answers; skip heading-heavy formal
reports unless he asks for depth; skip "want me to walk you through it?" offers — just give the
answer. He has flagged this MORE THAN ONCE — treat it as a hard rule, not a preference. Full
detail + the exact words I've slipped on before: memory `feedback_communication_style`.

## THE BAR — 100% accuracy + completeness — read every session, no exceptions
This is a project of accuracy and specifics. A wrong gloss/lemma/number misinforms a reader who
trusts it and can't check the Greek/Hebrew himself. The bar is NOT negotiable:
- **100% accuracy AND completeness.** Coverage (every word has *a* value) is NOT quality (the
  value is *right*) — prove BOTH before calling anything done. "Good enough", "basically done",
  "mostly covered" are rejected. Measure the gap completely (every row, no sampling), drive it
  to zero, re-verify.
- **Do NOT ship half-baked work to "get the job done."** If I catch myself writing a hedge —
  "it can read a bit X", "KJV-flavored", "good enough for now", "we can refine later" — STOP.
  That hedge means it isn't validated. Validate the SOURCE'S QUALITY on real samples BEFORE
  building on it; never ship a source I've already doubted and plan to "check after." The check
  comes before the commit, not after he pushes back. (2026-06-22: I shipped the word-card gloss
  on `kjv_def` after flagging it risky — exactly the failure this rule exists to stop.)
- **Never suggest an AI prompt change that REGRESSES the model** (parrots framing, over-asserts,
  adds jargon, blacklists instead of reframing). See memory `project_ai_synthesis_quality`.
- **Don't create work that we'll have to come back and fix.** Anything less than
  correct-and-complete is not a shortcut, it's a future bug.
- **If I'm about to propose or ship something less than this:** stop, run `/wrap`, and write a
  clean hand-off prompt for a fresh session instead of limping forward. Full record: memory
  `feedback_accuracy_completeness_bar`.

## VERIFY BEFORE YOU CLAIM — read every session, no exceptions
Every factual claim about the code/data/sources gets CHECKED against the actual source BEFORE I
say it — read the file+line, run the read-only check, look at the real rows. Inferring from one
or two lines and stating it as fact is banned. (2026-06-22: I claimed the ABP word card "shows a
dictionary gloss" — twice — without tracing the display logic; it shows the in-verse English
word in the common case.)
**No clean-sweep claims:** never call something a "clean sweep / sure win / validated / stark
improvement" from a flattering sample — check EVERY item or label it a spot-check with the rest
unverified, and probe STRESS cases (loaded/edge), not easy ones. (2026-06-22: I called TBESG
"validated" off love/spirit/God; χάρις→"grace" — a loaded one-word gloss — broke it. Reporting a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lexica-bible/lexica](https://github.com/lexica-bible/lexica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
