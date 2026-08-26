---
trigger: always_on
description: Instructions for AI coding agents (Claude Code, Codex CLI, and any other tool that reads this
---

# AGENTS.md

Instructions for AI coding agents (Claude Code, Codex CLI, and any other tool that reads this
convention) working in this repository. `CLAUDE.md` is a one-line pointer here — the same
AGENTS.md-canonical convention students follow in their portfolio repos.

## Working Principles

Adapted from [Andrej Karpathy's coding guidelines](https://github.com/multica-ai/andrej-karpathy-skills) for this docs/Markdown repository:

- **Think before editing** — state assumptions and surface ambiguity instead of guessing silently; offer the simpler option when one exists.
- **Simplicity first** — do the task that was asked; no speculative restructuring of courses, templates, or decision memos.
- **Surgical changes** — touch only what the request needs, and preserve each file's existing style and formatting (existing template conventions always win). Flag stray issues you notice rather than fixing them unasked; when records *document* a past change (e.g. a decision memo), update live materials, not the record.
- **Goal-driven** — define what "done" looks like, then verify it: `recalc.py` returns 0 errors, generated Office files validate, referenced paths resolve, no broken links.

## Repository Purpose

Unified portfolio and course materials hub for Shidler College of Business (University of Hawaiʻi at Mānoa) courses taught by Adam W. Stauffer. Contains syllabi, assignment frameworks, project templates, branded materials, and professional portfolio documents — all managed via Git/Markdown.

Student-facing tutorials live on the companion **Kumu site**, <https://adamwstauffer.github.io/ai-lms/> (published from the sibling `ai-lms` repo). This repo's stage briefs and the site's stage pages are kept in sync — a change to one usually implies a change to the other.

## Repository Structure

- **`courses/`** — Subject-first directories (e.g., `courses/International-Corporate-Finance/`), not course-code-first. See `courses/README.md` for the Shidler-code-to-directory map. Each subject directory shares one shape: `projects/<slug>/` (shared curriculum) + one `<CODE[-POPULATION]>/` subfolder per offering (e.g., `BUS-629-VEMBA/`, `FIN-321/`). See `docs/decisions/2026-07-08-generic-course-directory-naming.md` for the full rationale.
- **`docs/`** — Centralized documentation hub:
  - `_branding/` — UH Mānoa design tokens (`design.json`) and visual reference (`design-system.html`)
  - `templates/` — Reusable assignment templates (memo, spec, case brief, risk memo, prompt log)
  - `decisions/` — Strategic decision memos, flat (`YYYY-MM-DD-<slug>.md`; course-specific ones are `YYYY-MM-DD-<course-code>-<slug>.md`, e.g. `2026-05-07-bus629-stage2-restructure.md`)
  - `ai-usage-guidelines.md`, `writing-style-guide.md`, `reproducibility-playbook.md`
- **`BIO.md`** — Single source of truth for instructor biography; course READMEs link here
- **`scripts/`** — Repo-level tooling scripts; spreadsheet cleanup pipelines live in `scripts/spreadsheets/`

## Local-only trees — PII and history (hard rules)

Several root directories exist on Adam's machine but are **fully gitignored — they must never
appear on the public tree**, not even as placeholder READMEs:

- **`/rosters/`** — rosters, attendance sheets, approved-access lists (incl. the Kumu-site access list `rosters/approved/kumu-approved.xlsx`). Student PII: names, emails, IDs. FERPA — never commit anything from this tree, never add whitelist exceptions.
- **`/recommendations/`** — recommendation letters and the résumés, statements, and correspondence supporting them. One folder per student, `recommendations/<YYYY-MM>-<lastname>-<firstname>/`. Same rule: nothing here is ever committed.
- **`/_archive/`** — deprecated/historical course materials (e.g. `_archive/bus314/`, the archived BUS-314 project). Untracked 2026-08-06 to declutter the public repo; pre-removal contents remain in public git history.
- **`<CODE[-POPULATION]>/ignore/`** — per-offering student submissions and grading records.

Each local tree carries its own untracked `README.md` describing its layout — read it before
filing anything there. When work touches these trees, reference them by path in commits/memos but
never `git add` their contents.

### Within each subject directory

- `README.md` — Subject hub: overview, course-code table, links to `projects/` and offering folders
- `projects/<slug>/` — Shared curriculum: stage assignment docs, `_templates/`, `_tools/` (grading scripts), analysis/deliverables/models as applicable
- `<CODE[-POPULATION]>/README.md` — Per-offering syllabus (overview, objectives, grading, AI policy, campus policies)
- `<CODE[-POPULATION]>/ignore/` — Gitignored student submissions and grading records for that offering

## Project Workflow

Most projects follow a reusable pedagogical pattern. The default is five stages:

1. **Memo** (Stage 1) — Executive summary and problem framing
2. **Specification** (Stage 2) — Technical planning, methodology, pseudocode
3. **Excel Build** (Stage 3) — Quantitative/financial model in Excel
4. **Prompt Engineering** (Stage 4) — AI integration and prompt documentation
5. **Final Recommendations** (Stage 5) — Synthesis and actionable insights


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adamwstauffer/shidler](https://github.com/adamwstauffer/shidler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
