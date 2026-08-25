---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this system does

An AI-assisted job application pipeline with a two-stage funnel:

1. **Lead capture → triage**: Browser extension sends raw job board text to the backend instantly. Batch extraction (`POST /api/leads/extract-captured`) parses company/title/JD via LLM. Each lead is then analyzed (fit score + company tone) and either approved (→ Application) or rejected.
2. **Application pipeline**: Given a job description, produces a tailored one-page CV and cover letter (DOCX + PDF) stored under `applications/[Company]/`.

Initial setup: Parse a raw resume into `resume_master.md` / `resume_master_de.md`.

The full step-by-step orchestration is in `workflows/end-to-end.md`.

## Workflow permission protocol

Before starting any multi-step workflow (process captured jobs, generate CV/cover letter, interview prep), list every command you plan to run — reads AND writes — grouped by type, and ask once before doing anything. Example format:

**Read (auto-approved):** GET /api/leads/, resume_master.md, data/skills.json  
**Write (will prompt):** PUT /api/leads/{id}/processed ×3, WebSearch ×3

After the user says proceed, run everything. Write-operation prompts will still appear from the permission system — the user can click "Allow for this session" on the first occurrence of each pattern to avoid repeated prompts.

## Session-bounding convention

Long repetitive workflows degrade as the context window fills — the agent drifts from the rules it read at the top of the session. Bound them. For **any** workflow that processes many similar items (captured-job triage, skills extraction, multi-application generation), do this:

1. Process at most **N items per pass** (default N=4; the count is per-workflow and may be tuned).
2. After the batch, **STOP** — do not continue past N in the same pass. Report which items were processed and how many remain.
3. Tell the user to `/clear` and re-paste the prompt to continue the next batch, so each batch runs in a fresh context that re-reads these rules.

The point is to bound context/token usage per run and keep the last item's output as faithful as the first. For genuinely small counts a single all-at-once pass is fine (e.g. plain "process my captured jobs" with only a few pending) — the bound matters when the item count is large. Script-driven workflows (`scripts/batch_generate.py`) express the same bound as a `--limit` flag plus a printed resume cursor rather than a `/clear` instruction.

This convention generalizes the captured-jobs batch mode (see "Batch mode" under the leads pipeline below) to every repetitive workflow.

## Key commands

```bash
# Start the web app (both must run simultaneously)
cd app/backend && source ../../.venv/bin/activate && uvicorn main:app --reload   # :8000
cd app/frontend && npm run dev                                                   # :3000

# Install backend dependencies (first time)
source .venv/bin/activate && pip install -r app/backend/requirements.txt

# Unpack/pack DOCX for XML editing
python app/backend/office/unpack.py <template.docx> <output_dir>/
python app/backend/office/pack.py <unpacked_dir>/ <output.docx> --original <template.docx>

# Convert DOCX to PDF (requires LibreOffice)
libreoffice --headless --convert-to pdf <file.docx> --outdir <outdir>/

# Check page count of a PDF
pdfinfo <file.pdf> | grep Pages
```

## Web App (`app/`)

A local Next.js + FastAPI app that replaces the manual Claude Code workflow with a structured UI. It enforces all guardrails in code so the LLM cannot exaggerate.

- **Backend**: `app/backend/` — FastAPI + SQLite. Routers: `/api/resume`, `/api/application`, `/api/tracker`, `/api/settings`, `/api/leads`, `/api/trash`. Services: `generator.py` (locked tailoring + streaming), `reviewer.py` (persona + 2 random reviewers), `researcher.py` (company scraper + tone classifier), `analyzer.py` (JD gap analysis), `interview.py` (prep + skills debrief), `pdf.py` (LibreOffice + 1-page check).
- **LLM layer** (`services/llm.py` + `services/providers/`): `generate(model, prompt, system, fmt)` dispatches on a `provider/model` slug. `fmt` is an optional JSON schema for structured output, translated per provider: Ollama → `format` (full schema); Anthropic → forced tool call (`input_schema=fmt`, guaranteed schema-valid); OpenAI → `response_format` json_schema; Gemini → `responseMimeType: application/json` (valid-JSON floor only — Gemini's responseSchema dialect rejects the `$defs`/`additionalProperties` the Pydantic schemas use). Perplexity accepts `fmt` but ignores it (json_schema is gated/beta). Because not every provider enforces the schema, callers using `fmt` must still keep their JSON-sanitizing / graceful-fallback path. Output schemas live next to their service (`analyzer_schema.py`, `researcher_schema.py`, `reviewer_schema.py`, `skill_extractor_schema.py`, `interview_schema.py`). Streaming (`stream()`) does not support `fmt`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anahtiris/job-application-system](https://github.com/anahtiris/job-application-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
