---
trigger: always_on
description: **Project:** DataLineage Doctor
---

# AGENTS.md

**Project:** DataLineage Doctor
**Version:** 1.0
**Date:** April 19, 2026

---

## Purpose

This file is the **entry point for every AI coding assistant** working on this project. It is placed at the repository root so that AI tools (Cursor, Windsurf, GitHub Copilot Workspace, Claude, GPT-4, Gemini, etc.) find it automatically on session start.

Read this file completely before reading any other file or writing any code.

---

## Step 1 — Read the Status File First

```
knowledge/agent-sync/ai-project-status.md
```

This is the single source of truth for where development currently stands: current sprint, ticket statuses, what is done, what is in progress, what is blocked. **Do not skip this step.** Every other action depends on knowing where the project is.

---

## Step 2 — Read the Rules File

```
knowledge/agent-sync/ai-rules.md
```

This defines all non-negotiable rules: locked architectural decisions, module boundary constraints, file placement rules, code style rules, test rules, and what is out of scope. Violations are not acceptable regardless of how reasonable they may seem.

---

## Step 3 — Orient with the Project Summary

```
knowledge/project-understanding/project-summary.md
```

A full orientation to the project — what it does, why it exists, how the 5 modules relate, the tech stack rationale, the 5-sprint plan, and what documents to read for specific questions. Read this if you are new to the project or returning after a gap.

---

## Step 4 — Read the Relevant Service Doc Before Touching Any File

Before modifying any file, read the service doc that owns it:

| File location | Read this service doc |
|---|---|
| `app/` | `knowledge/services/app.md` |
| `agent/` | `knowledge/services/agent.md` |
| `worker/` | `knowledge/services/worker.md` |
| `om_client/` | `knowledge/services/om-client.md` |

---

## Project Structure

```
datalineage-doctor/
│
├── AGENTS.md                          ← You are here
│
├── app/                               # FastAPI HTTP server
│   ├── main.py                        # App factory + lifespan
│   ├── config.py                      # pydantic-settings — ONLY env var reads
│   ├── database.py                    # SQLAlchemy async engine + session factory
│   ├── dependencies.py                # FastAPI Depends() helpers
│   ├── models/                        # SQLAlchemy ORM models
│   ├── schemas/                       # Pydantic request/response schemas
│   ├── routers/                       # One file per route group
│   └── services/                      # Business logic (incident_store, metrics)
│
├── agent/                             # LLM reasoning loop
│   ├── loop.py                        # run_rca_agent() — the tool-calling while loop
│   ├── prompts.py                     # SYSTEM_PROMPT + build_user_message()
│   ├── parser.py                      # Parses LLM final message → RCAReport
│   ├── notifications.py               # Slack + OM incident creation
│   ├── tools/                         # Tool handlers + registry
│   │   ├── registry.py                # RCA_TOOLS list + TOOL_HANDLERS dict + dispatch()
│   │   ├── lineage.py                 # get_upstream_lineage, calculate_blast_radius
│   │   ├── quality.py                 # get_dq_test_results
│   │   ├── pipeline.py                # get_pipeline_entity_status
│   │   ├── ownership.py               # get_entity_owners
│   │   └── history.py                 # find_past_incidents (queries local DB)
│   └── schemas/
│       ├── report.py                  # RCAReport, TimelineEventInput, BlastRadiusConsumerInput
│       └── tool_outputs.py            # Typed return shapes per tool
│
├── worker/                            # Celery task management
│   ├── celery_app.py                  # Celery app instance + config
│   ├── tasks.py                       # rca_task() — the only task in MVP
│   └── persistence.py                 # save_incident() two-phase DB write
│
├── om_client/                         # OpenMetadata REST API client
│   ├── client.py                      # OMClient — async context manager, auth, _get()/_post()
│   ├── lineage.py                     # get_upstream_lineage(), get_downstream_lineage()
│   ├── quality.py                     # get_dq_test_results()
│   ├── pipeline.py                    # get_pipeline_status()
│   ├── ownership.py                   # get_entity_owners()
│   ├── incidents.py                   # create_incident()
│   └── schemas/                       # Typed OM response models
│
├── dashboard/                         # Web UI
│   ├── static/style.css               # Nexus design tokens + base styles
│   └── templates/
│       ├── base.html                  # Layout template
│       ├── incidents_list.html        # GET / — incident list
│       └── incident_detail.html       # GET /incidents/{id} — detail + React Flow graph
│
├── tests/                             # pytest test suite
│   ├── conftest.py                    # Fixtures: mock_om_client, async_client, db session
│   ├── test_webhook.py
│   ├── test_agent_loop.py
│   ├── test_tools_lineage.py
│   ├── test_tools_quality.py
│   ├── test_tools_pipeline.py
│   ├── test_tools_ownership.py
│   ├── test_tools_history.py
│   ├── test_om_client.py
│   └── test_incident_store.py
│
├── scripts/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mahboob-A/datalineage-doctor](https://github.com/Mahboob-A/datalineage-doctor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
