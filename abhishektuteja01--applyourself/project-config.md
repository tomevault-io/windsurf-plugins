---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal, human-gated job-search pipeline, sponsorship-aware throughout (the
scoring pre-screen and the outreach disclosure rules both know about it). It scrapes
jobs, scores them against a profile, and generates tailored application material.
Every outward action (applying, sending outreach) is gated on the user; the code
never submits anything.

## Instructions

1. No dated references if not needed, no extra verbose paragraphs explaining why the decision was made. If absolutely needed (in case of A/B tests and similar), a concise one liner is enough.
2. Explain code changes and tests before updating or implementing. Start post confirmation.

## The one rule that shapes everything: the determinism boundary (R7)

**No module under `src/` ever calls an LLM.** `src/` is deterministic plumbing:
parquet I/O, config loading, cleaning, linting, docx rendering, state transitions.
All *judgment* (scoring a job, tailoring a resume, writing a cover letter/outreach)
happens inside a slash-command session in `.claude/commands/*.md`, which calls the
`src/` helpers via Bash for the deterministic parts. When editing, keep judging out
of `src/` and keep parquet/state mutation out of the command prose.

A corollary: `src/` is **vertical-agnostic and company-agnostic**. Never
hardcode a vertical name, search term, or company. Those come only from
`profile/*.yaml` and `data/universe/*.csv`.

**R7 and R10 are the only rule codes in this repo.** Every other rule is stated
inline where it applies, by name (`NO-FAB`, `NO-DRIFT`) or in plain words.

## Pipeline stages (data flow)

1. **Discovery** (`src/discovery/`, CLI `discover`) — deterministic, LLM-free
   overnight scrape. Sources in order: manual `inbox/*.md` clips → JobSpy
   (LinkedIn + Indeed) → Greenhouse/Lever/Ashby JSON boards over
   `data/universe/*.csv` + `profile/companies.yaml`. Board/inbox rows are
   title-classified into a vertical at fetch time; unclassified rows dropped.
   Always ends by running cleaning (try/finally), even after a crash/deadline.
2. **Cleaning** (`src/discovery/cleaning.py`) — normalize, drop short/stale rows,
   drop rows outside the location allowlist, dedupe (exact then rapidfuzz
   WRatio ≥ 90), assign `job_id`, tag seen-ledger. Writes `jobs/clean.parquet` (the **only** discovery
   output downstream reads) + `clean.preview.jsonl`.
3. **Scoring** (`/score`, `/rescore`; plumbing in `src/prescreen.py` for the
   deterministic pre-screens, `src/scoring_io.py` for parquet read/dump/merge/
   prune, `src/shortlist.py` for compute+render, and `src/score_cli.py`) — LLM judges rows and writes `jobs/scored.parquet` +
   `shortlist/<date>.md`. See below.
4. **Application material** (`/tailor`, `/cover-letter`, `/outreach`) — generate
   docx/pdf into `applications/<vertical>/<dir>/`; docx rendering via
   `src/docx_render.py` (resume) and `src/docx_cover_letter.py`
   (placeholder fill); text passes `src/lint.py`.
5. **Tracking** (`/track`, `/standup`; plumbing in `src/state_io.py`,
   `src/track_cli.py`) — one `pipeline/<job_id>/state.yaml` per role moving
   through an 11-state machine.

### `job_id` is a content hash — treat it as load-bearing

`job_id = sha1(company_normalized + "|" + title_normalized)[:8]`. URL and
`jd_text` are deliberately **excluded** so the id is stable across re-scrapes.
Changing the hash inputs would silently orphan `pipeline/<job_id>/state.yaml` and
`applications/<dir>` keys. Do not add url/jd_text to the hash.

## Verticals: the config spine

A "vertical" is a job lane. `src/verticals.py`
is the single source of truth loader.

- Config lives in `profile/verticals.yaml` (gitignored user data) with a matching
  `profile/verticals/<name>/{rubric.md, tailoring.md}` dir per vertical, plus the
  resume each block's required `resume_file` points at (judges score against it,
  per `score-judge.md`). `verticals-check` fails loud if any of the three is missing.
- The loader is **strict**: every vertical block must have all current required
  keys or it raises `ValueError`. Because `tests/conftest.py` injects the config
  via an autouse fixture, a malformed block errors the *entire* test suite.
- **Two fixture mirrors must stay byte-identical to each other** for tests to
  pass: `tests/fixtures/verticals.yaml` and `tests/discovery/fixtures/verticals.yaml`.
  Any schema change must be mirrored into both in the same change.
  `TestFixtureMirrors` enforces it.
- Consumers must call `verticals.get_config()` **inside function bodies**, never at
  module level, so test injection via `set_config()` always wins.
- Templates for onboarding a new vertical: `profile/*.example.yaml` and
  `profile/verticals/example_*/` (three: primary, secondary, tertiary — the
  fixtures' `default_vertical` is tertiary). Use `/new-vertical`.

> The two `tests/**/fixtures/verticals.yaml` files are **synthetic** — three
> fictional verticals (`example_primary/secondary/tertiary`), no real search
> terms or skill weights. The real config is covered separately by
> `tests/test_real_config_drift.py`, which skips when `profile/verticals.yaml`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abhishektuteja01/ApplYourself](https://github.com/abhishektuteja01/ApplYourself) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
