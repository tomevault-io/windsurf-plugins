---
trigger: always_on
description: **What this repo is.** A public, field-scoped toolkit of Claude Code skills for researchers in **language assessment / applied linguistics / EAP writing research**. Each skill packages a complete, integrity-first workflow; Claude operates the machinery, the scholar keeps the judgement calls. Maintainer: Sha Liu. License: MIT (except `starter-kit/style.csl`, CSL project, CC BY-SA — noted in LICENSE).
---

# CLAUDE.md — Claude Language Assessment Research Kit (CLARK)

**What this repo is.** A public, field-scoped toolkit of Claude Code skills for researchers in **language assessment / applied linguistics / EAP writing research**. Each skill packages a complete, integrity-first workflow; Claude operates the machinery, the scholar keeps the judgement calls. Maintainer: Sha Liu. License: MIT (except `starter-kit/style.csl`, CSL project, CC BY-SA — noted in LICENSE).

## Layout
- `skills/zotero-citations/` — the citation pipeline operator (Zotero → Better BibTeX → Pandoc → Word); guided onboarding for users who have never used any of the tools.
- `skills/citation-integrity/` — standalone pre-submission reference audit: DOI resolve-and-match, retraction check, preprint→version-of-record check (`starter-kit/resolve_check.py` · `check_retractions.py` · `vor_check.py`); enforces `docs/04`.
- `skills/literature-review/` — living review matrix (approval-gated triage; matrix = source of truth) + generated Obsidian views via `starter-kit/matrix_to_vault.py`; per-project rules live in that project's `review-conventions.md` (template in starter-kit); optional cross-vendor row audit (`starter-kit/review_audit.py`).
- `skills/literature-radar/` — the standing watch: scheduled/on-demand sweeps → dedup (matrix + folder + digests + Zotero) → pre-screened triage-ready digest (discovery), plus Mode C — the corpus-state watch over papers already held (corrections · retractions · versions of record; index check + held-PDF notice scan + publisher-page glance); propose-only, feeds literature-review Mode 1.
- `skills/pre-submission-review/` — evidence-anchored evaluation of the USER'S OWN manuscript only: two-pass evidence discipline (ledger with PASS/FAIL recomputation, claim–warrant checks, triage gate), revision plan + 0–100 estimate with readiness verdict, live author-instructions compliance, fresh-agent verification, optional cross-vendor audit (`starter-kit/manuscript_audit.py`, gated by `--confirm-send`). `references/` = the two article-type criteria modules (built from publisher/journal-provided reviewer material) + what major venues tell their reviewers to look for.
- `skills/writing-polish/` — five-dimension polish of the user's own prose (accuracy corrected directly with a change log; everything stylistic propose-only), built for multilingual and early-career academics.
- `skills/model-radar/` — the maintenance watch on pinned audit models: pinned-ID liveness (highest priority: dead/retiring pins), vendor prompting-guide deltas, release/price changes → one propose-only log entry per run; keeps the audit layers reproducible.
- `docs/00–09` — the user-facing operator's manual (project wiring + session hygiene, setup, daily use, troubleshooting, the integrity charter, the Zotero-MCP upgrade, Obsidian-as-editor, the living literature review, the pre-submission review) + the flowchart twins (`flowchart.md` — Mermaid, renders on GitHub; `flowchart.html` — styled, printable; keep them in sync) + `images/`.
- `starter-kit/` — the copy-into-your-project kit: `render.sh`/`render.bat`, `style.csl` (APA 7th default, swappable), sample `library.bib` + draft, and the stdlib-only Python tools (`resolve_check` · `check_retractions` · `vor_check` · `pdf_probe` · `notice_scan` · `matrix_to_vault` · `review_audit` · `manuscript_audit` · `clark_doctor`).
- `field-guides/` — the **CLARK Field Guides** series: expert-curated, citable domain packs (data, not code) consumed by the skills — `README.md` is the spec (structure · provenance rules · the three contribution rungs A/B/C · registry), `_template/` the starting kit. Governance: a guide extends the specifics layer ONLY (never the skills, never docs/04); provenance rules are charter-grade (published sources only, point-and-cite never republish, written consent for anything unpublished, last-verified dates); acknowledgments never imply endorsement without the person's explicit agreement on wording; editorial gate = the maintainer.
- `_private/` — the maintainer's records, git-ignored, never published. **Maintainer sessions: if `_private/CLAUDE-maintainer-log.md` exists, read it at session start** — it carries the working history, state, and roadmap. (In a downloaded copy of this repo that file doesn't exist; ignore this line.)

## Non-negotiable design rules
1. **`docs/04-citation-integrity.md` is the single source of truth for the 16 integrity rules** (COPE/ICMJE-aligned). Skills reference it; never fork or restate the rules elsewhere.
2. **Flag, never guess; propose, never apply.** No skill or script invents, "remembers," or silently corrects a reference. Fixes are proposed with evidence; the user applies them in Zotero. Scripts only ever *report* (exit codes for gating).
3. **An AI may format and check citations — it may never supply one.** Reading, choosing, and interpreting sources stay human.

## Doc-writing standard (all user-facing docs)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shalice-in-AIland/claude-language-assessment-research-kit](https://github.com/Shalice-in-AIland/claude-language-assessment-research-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
