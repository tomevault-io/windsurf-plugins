---
trigger: always_on
description: > **How to use this file.** Paste its contents (or point your agent at it) from a fresh
---

# AGENTS.md: onboarding for a coding agent

> **How to use this file.** Paste its contents (or point your agent at it) from a fresh
> clone of this recipe. It tells the agent what the project is, the invariants it must not
> break, and the exact sequence to take it from clone → running, including wiring up the
> user's own CRM. It's written to be executed, not just read.

---

## Your task, in one line

Get **Investor Signals + Sales Enrichment** running for the user: install everything,
collect the few things you can't invent (their Parallel API key, their fund watchlist, and
which CRM they use), launch the app, wire up their CRM, and, if they want it, create the
investor monitors and run the first sweep. Follow the steps in order; pause only for the
inputs marked **ASK THE USER**.

## What this project is

Two workflows on the Parallel **Task** and **Monitor** APIs, sharing one Python core:

1. **Investor Signals pipeline** (`monitor/`), one daily Parallel Monitor per fund on the
   user's watchlist detects new AI-native funding rounds (seed–Series B); each detection is
   verified by a chained follow-up Task (`previous_interaction_id`), scored by a priority
   policy, optionally checked against the user's CRM, and delivered to Slack (weekly digest
   or real-time webhook).
2. **Sales Enrichment app** (`project/`), a React + FastAPI app. Type a company → two
   concurrent Task runs (account + contacts) merge into one cited `ResearchBrief`. Bulk CSV
   mode and an ad-hoc "ask bar" too.

**Stack:** FastAPI (Python ≥ 3.12) backend, Vite + React + TypeScript frontend, deployable
to Vercel as one serverless function + static build. `parallel-web` SDK.

## Map of the code

| Path | What it is |
|---|---|
| `project/backend/parallel_client.py` | ★ The **only** file that reads `PARALLEL_API_KEY` and calls the Task API. |
| `project/backend/investor_core.py` | ★ Shared core: qualification schema, priority policy, Slack Block Kit formatting. One source of truth for the CLI, the app, and the webhook. |
| `project/backend/crm.py` | ★ The **CRM adapter seam**: selects a provider and normalizes its result. This is where you plug in the user's CRM. |
| `project/backend/attio_client.py` | The reference CRM adapter (Attio). Copy it to add another CRM. |
| `project/backend/main.py` | FastAPI routes, the access gate, the webhook receiver, the cron endpoint. |
| `project/backend/signals_service.py` | Signals surfaces + the weekly-digest job. |
| `project/src/` | The React app. `types.ts` mirrors the backend `ResearchBrief` exactly. |
| `monitor/config.py` | Loads the watchlist + holds the monitor queries/schemas/processors. |
| `monitor/investors.example.json` | Sample watchlist. Copy to `investors.json` (gitignored). |
| `monitor/{sweep,monitors,check,slack_notify,build_portfolio}.py` | The pipeline CLI. |
| `tests/` | pytest suite (credibility gate, priority policy, routes, Slack blocks). |

## Invariants: do not break these

1. **The API key stays server-side.** Only `parallel_client.py` (and the async webhook in
   `main.py`) touch `PARALLEL_API_KEY`. The browser talks exclusively to `/api/*`. Never
   put the key in frontend code, logs, or anything committed.
2. **Never fabricate a value.** The credibility rule is load-bearing: any field without a
   supporting citation is returned as `null`. Don't "helpfully" fill blanks.
3. **Don't touch the output JSON schemas or the `field-basis` beta header** in
   `parallel_client.py` without updating `types.ts` and `to_research_brief()` in lockstep,    the field names map one-to-one to the UI.
4. **Secrets and private data never get committed.** `.env`, `monitor/investors.json`,
   `data/`, and the generated `monitor/*.json` (`portfolio_names.json`, `monitors.json`,
   `signals.json`, `state.json`) are all gitignored. Keep them that way. Use the
   `.example` files for anything you commit.
5. **The watchlist and the CRM belong to the user.** There is no default fund list, and no
   CRM is assumed, ask.

## Setup sequence

Run these from the recipe root (`python-recipes/parallel-investor-signals`).

### Step 1: install (safe to run unattended)

```bash
make setup     # creates the venv, installs backend + dev + frontend deps, scaffolds .env
```

### Step 2: collect what you can't invent

**ASK THE USER** for:

- **Their Parallel API key** (from [platform.parallel.ai](https://platform.parallel.ai)).
- **The VC funds they want to track**, by press name (e.g. "Sequoia Capital",
  "Andreessen Horowitz"). If they don't have a list yet, use the example list and tell them
  they can edit `monitor/investors.json` later.
- **Which CRM they use** (Attio, HubSpot, Salesforce, Pipedrive, Affinity, none, …). You'll
  wire it up in Step 4.

Write the key and watchlist (do **not** echo the key back or commit `.env`):

```bash
# .env: set at least these two
#   PARALLEL_API_KEY=<their key>
#   DEMO_PASSWORD=<any passphrase; the app gate is closed until this is set>

cp monitor/investors.example.json monitor/investors.json
# then edit monitor/investors.json → the "investors" array = their funds
```

Confirm the backend can see the key without printing it:

```bash
source project/backend/.venv/bin/activate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [parallel-web/parallel-cookbook](https://github.com/parallel-web/parallel-cookbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
