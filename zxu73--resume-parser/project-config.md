---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BetterCV — an AI-powered resume optimization platform. Users upload a `.docx` resume and paste a job description, then a multi-agent pipeline evaluates fit, rates bullets, suggests rewrites, and optionally recommends experience swaps from a pool.

## Architecture

- **Frontend**: React 19 + TypeScript + Vite, styled with Tailwind CSS and Radix UI components. Path alias `@/*` maps to `./src/*`.
- **Backend**: FastAPI + LangGraph for multi-agent orchestration. Agent nodes call OpenAI via `langchain_openai.ChatOpenAI` with `.with_structured_output(...)`; the `/chat` and `/job-search` endpoints instead call LiteLLM directly with `acompletion` for lightweight, non-graph completions (freeform chat, JD query extraction). Model is configurable via the `REASONING_MODEL` env var (defaults to `gpt-4o-mini`).
- **Document handling**: python-docx for `.docx` manipulation, PyPDF2/pymupdf for PDFs.

### Agent Pipeline (sequential)

1. **Evaluation** (`evaluate_only_graph`, single node) — exhaustive keyword extraction, STAR analysis, produces `EvaluationResponse` (scores, strengths, weaknesses, missing skills) plus **anchored clarifying questions**.

   Question count is capped by `max_length=8` on the schema field, and the cap is load-bearing: the model sits at whatever ceiling it is given (at the old 3-5 it almost always returned 3). But count and pairing quality are directly opposed, and the prompt cannot hold both. Measured on a 12-bullet resume with no ML experience against an ML-heavy JD: at "aim for 8" it hit 8 nearly every run with only ~40% of pairings plausible — it runs out of honest anchors and starts attaching domain skills to unrelated bullets (`embedding model selection` ← "Wrote unit tests for the payments module"). Removing the floor entirely swung the other way, to 1-5 questions. The current wording (target 6, ceiling 8, floor 5) lands at 3-7 with ~60% plausible; the runs that come back with 3-4 are the ones where nearly every pairing is good.

   A bad pairing is not cosmetic. A "yes" on one is treated as ground truth by `_confirmed_block`, which validates only that the anchor exists in the resume, never that the skill fits it — so it goes straight into a rewrite the candidate has to defend in an interview. The `PLAUSIBILITY GATE` section of `EVALUATION_INSTRUCTION` is what pushes back, including a ban on the hedging phrasings ("did you consider…", "any techniques related to…") that are the tell that a pairing failed. It helps and does not fully hold.

   Two rules are enforced in `evaluate_node` rather than the prompt, because the model ignored both in roughly half of runs and neither needs judgement: `skill_targeted` must be one of the model's own `missing_skills` (a coined phrasing like "shipping LLM-backed features" is not a JD keyword, so a "yes" buys no ATS hit while spending a question slot — and `remaining` filters `missing_skills` by exact string, so the coined skill never leaves the pool and gets placed a second time by Rule A), and at most one question may anchor to a given bullet (a second is waste: `_confirmed_block` groups by `target_bullet`, so both skills land in the one rewrite anyway). Both are at 100% after the change, from ~50%. Every `ClarifyingQuestion` carries a `target_bullet`: the verbatim resume line the question is about ("when you did X, did you use <missing skill>?"). `evaluate_node` runs each anchor through `_snap_to_resume`, which fuzzy-snaps a lightly-paraphrased anchor back to real resume text and blanks it otherwise — a near-miss anchor would produce a `current_text` the document editor can never match.
2. **Rating** (`rate_only_graph`, fanned out) — takes the user's answers and produces:
   - `keyword_suggestions`: rewrites that insert missing JD skills in STAR format. `keywords_added` must be non-empty, each keyword a literal substring of `suggested_text`.
   - `star_suggestions` (target 5-10): STAR-format improvements only. `keywords_added` must be `[]`.
   - Each bullet appears in at most one section. All section invariants are enforced in one place — `RatingResponse.normalize_sections` in `agent.py` — at pydantic validation time; `app.py` does no post-processing. Governed by anti-hallucination rules in `backend/src/agent/guidelines.md`.
3. **Experience Optimizer** (`optimizer_graph`, single node, optional) — scores resume vs pool experiences on JD fit, recommends 1-for-1 swaps only if pool score exceeds resume by 20+ points.

`/apply-swaps-docx` finds the end of an experience block via `starts_new_block`, which checks for a `Heading` style, bold text, a date range (`2021-2024`, `(2020)`, `Jan 2020 - Present`), or the transition out of a bullet list. The last two exist because job sub-headings are often plain text — with only the bold/Heading check the block ran on to the next real heading (`Skills`) and blanked every job in between. The date and list-transition signals are deliberately skipped for list paragraphs, so a bullet that mentions a year or bolds a word is not mistaken for a heading. The endpoint returns `swaps_applied` / `swaps_failed`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zxu73/resume-parser](https://github.com/zxu73/resume-parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
