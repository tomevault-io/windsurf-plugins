---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**Install app into a Frappe bench:**
```bash
bench get-app <REPO_URL> --branch develop
bench install-app deloitte
```

**Linting:**
```bash
cd apps/deloitte
pre-commit install       # set up hooks (run once)
pre-commit run --all-files  # run all checks manually
```

Individual tools:
```bash
ruff check .             # lint Python
ruff format .            # format Python
```

**There are no automated tests in this repository yet.**

## Architecture

This is a [Frappe Framework](https://frappeframework.com) app that extends the Frappe/ERPNext `Job Candidate` DocType to sync candidate data to Google Sheets via a Google Apps Script Web App.

### Key files

- **`deloitte/hooks.py`** — Frappe entry point. Registers the `on_update` doc event for `Job Candidate`, wiring it to `sync_sheet.on_job_candidate_update`.
- **`deloitte/scripts/sync_sheet.py`** — All business logic. Two public functions:
  - `on_job_candidate_update(doc, method)` — Called automatically on every Job Candidate save. Reads fields from the doc and POSTs to the Apps Script endpoint.
  - `sync_job_candidates_to_sheet(candidates)` — Batch sync; takes a list of candidate names.
- **`deloitte/config/settings.py`** (gitignored) — Expected to export `APPS_SCRIPT_WEB_APP_URL`, the Google Apps Script endpoint URL.

### Full System Pipeline

The system spans two codebases: this Frappe app (`deloitte/`) and the standalone voice service (`/home/ubuntu/deloitte-voice-service/`).

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        DELOITTE INTERVIEW PIPELINE                              │
└─────────────────────────────────────────────────────────────────────────────────┘

  [HR / Admin]
      │
      │  Creates Job Candidate record in Frappe ERP (del.alfaedge.in)
      ▼
  ┌───────────────────────────────┐
  │  Frappe: Job Candidate DocType│
  │  (hooks.py doc_events)        │
  └──────────────┬────────────────┘
                 │
        ┌────────┴────────┐
        │  after_insert   │  on_update
        ▼                 ▼
  ┌───────────────┐  ┌──────────────────────────────────────┐
  │ send_whatsapp │  │ sync_sheet.on_job_candidate_update() │
  │ .py           │  │ → HTTP POST to Google Apps Script    │
  │               │  │   Web App (external)                 │
  │ Sends WhatsApp│  │ Payload:                             │
  │ template msg  │  │  • sheet1_updates (phone, score,     │
  │ to candidate  │  │    eligibility, ai_evaluation)       │
  │ via Meta      │  │  • job_tab_updates (Q&A per job)     │
  │ Graph API     │  └──────────────────────────────────────┘
  │ (external)    │                   │
  └───────┬───────┘                   ▼
          │              ┌────────────────────────┐
          │              │  Google Sheet (external)│
          │              │  Rows updated per       │
          │              │  candidate + job tab    │
          │              └────────────────────────┘
          │
          │  WhatsApp call-button template
          ▼
  ┌─────────────────────────────────────────────────────────────┐
  │                  CANDIDATE (WhatsApp)                        │
  │  Receives template message → taps "Join Interview Call"     │
  └───────────────────────────┬─────────────────────────────────┘
                              │  Incoming WhatsApp Voice Call
                              ▼
  ┌─────────────────────────────────────────────────────────────┐
  │           deloitte-voice-service  (FastAPI)                  │
  │           /home/ubuntu/deloitte-voice-service/               │
  │                                                              │
  │  POST /webhook  ← Meta Cloud API webhook (external)          │
  │  webhook.py                                                  │
  │    │                                                         │
  │    │  1. Parses call event (connect / terminate)             │
  │    │  2. Reserves call ID via state_manager                  │
  │    │  3. Fetches candidate details from Frappe               │
  │    │     GET del.alfaedge.in/api/method/                     │
  │    │         get_candidate_details_and_questions             │
  │    │     ← returns name, job, interview questions            │
  │    │                                                         │
  │    ▼                                                         │
  │  voice_agent.py  (Pipecat pipeline)                          │
  │    │                                                         │
  │    │  ┌─────────────────────────────────────────┐           │
  │    │  │  SmallWebRTC Transport (audio in/out)   │           │
  │    │  │    ↕ real-time audio with candidate     │           │
  │    │  └──────────────┬──────────────────────────┘           │
  │    │                 │                                       │
  │    │  ┌──────────────▼──────────────────────────┐           │
  │    │  │  Silero VAD → LLM Context Aggregator    │           │
  │    │  │  (turn detection + idle timeout)        │           │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alfaedge-crm/deloitte](https://github.com/alfaedge-crm/deloitte) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
