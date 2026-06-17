---
trigger: always_on
description: This file is the technical guide for the entire monorepo. End-user CLI docs live in `DOCS.md`.
---

# Oddish Repository Guide

This file is the technical guide for the entire monorepo. End-user CLI docs live in `DOCS.md`.

The repo has three main packages:

- `oddish/` — the core Python CLI, FastAPI server, queueing layer, and worker runtime
- `backend/` — the hosted cloud layer built on top of `oddish`; adds multi-tenant auth, Modal deployment, and product-specific endpoints
- `frontend/` — the Next.js App Router dashboard and public pages

Python `3.12+` is required for `oddish` and `backend`. Node.js `20+` and `pnpm` are required for `frontend`.

## Maintenance Notes

- Keep `oddish/README.md` focused on end-user CLI workflows.
- Put `oddish` implementation details, architecture notes, and local development guidance here.
- If you change the CLI surface in `oddish/src/oddish/cli/`, update `oddish/README.md`.
- If you change API contracts, queue behavior, or storage layout, update this file.
- If you change `backend/` auth, deployment, or worker orchestration, update this file.
- If you change `frontend/` routing, API proxy structure, or auth behavior, update this file.

## Repository Layout

```text
oddish/                         # Core Python package (CLI, server, workers, DB)
├── src/oddish/
│   ├── cli/                    # oddish run/status/cancel/pull/combine/delete
│   ├── core/                   # shared business logic (reused by backend/)
│   ├── server/                 # standalone FastAPI app (python -m oddish.server)
│   ├── db/                     # models, connection helpers, storage
│   ├── workers/                # Unified worker_jobs runtime: dispatcher,
│   │                           #   single-job runner, handlers, cleanup
│   ├── backfill_queue_keys.py
│   ├── config.py
│   ├── experiment.py
│   ├── queue.py                # task/trial enqueue + worker_jobs enqueue helpers
│   └── schemas.py
├── alembic/                    # Core DB migrations
├── env.example
└── pyproject.toml

backend/                        # Hosted cloud layer (Modal deployment)
├── api/
│   ├── app.py                  # FastAPI app factory and lifespan wiring
│   ├── schemas.py              # Pydantic models for org/auth/share responses
│   └── routers/                # tasks, trials, dashboard, orgs, api_keys, admin, webhooks
├── auth/                       # API key + Clerk JWT verification, provisioning, types
├── worker/                     # Modal dispatcher and single-job worker orchestration
├── deploy.py                   # Modal app entrypoint
├── modal_app.py                # Modal image, volumes, shared runtime
├── endpoints.py                # Modal ASGI app function with concurrency/volume wiring
├── serve.py                    # Railway/uvicorn entrypoint for non-Modal deployment
├── cloud_policy.py             # Hosted-only environment policy
├── models.py                   # Cloud auth models (orgs/users/api keys)
├── alembic/                    # Cloud migrations (auth + cloud table extensions)
└── pyproject.toml

frontend/                       # Next.js App Router dashboard
├── src/
│   ├── app/
│   │   ├── page.tsx            # Public landing page / signed-in redirect
│   │   ├── (app)/              # Authenticated app shell (dashboard, tasks, experiments, settings, admin)
│   │   ├── share/[token]/      # Public experiment page
│   │   ├── datasets/           # Public dataset pages
│   │   ├── api/                # Backend proxy route handlers
│   │   └── providers.tsx       # Shared SWR config
│   ├── components/             # Dashboard, detail panels, charts, nav, UI primitives
│   ├── lib/                    # API helpers, backend config, shared types, utilities
│   └── middleware.ts           # Clerk route protection
└── package.json
```

## System Architecture

```text
Browser / oddish CLI
        |
        v
Next.js route handlers (frontend/src/app/api/*)
        |
        v
FastAPI server — oddish standalone (python -m oddish.server)
           or backend cloud layer (Modal / Railway)
        |
        v
Postgres
  - worker_jobs       # unified queue (TRIAL / QA / …)
  - trials / tasks    # domain state + live UI columns
  - queue_slots       # per-queue-key concurrency leases
        |
        v
Workers (auto-started by API, or standalone via python -m oddish.workers.queue.worker)
        |
        v
Harbor task execution → logs/results/artifacts (S3)
```

High-level flow:

1. Upload a task bundle directly to S3 via a presigned PUT URL.
2. Submit a sweep of agent/model trials for that task; each trial is
   enqueued as a `worker_jobs` row in the same transaction as its domain
   row. Set `max_trial_attempts` on a sweep submission or sweep config to
   override the total attempt budget for newly-created trials.
3. Workers claim one `worker_jobs` row at a time, dispatch to the registered
   handler (`TRIAL` / `QA`), write heartbeats, and exit.
4. Trajectory analysis is **task-scoped**: when every trial of a
   `run_analysis` task is terminal, a single `QA` job is enqueued. That one
   job classifies every live trial's trajectory (same taxonomy / evidence /
   reasoning, written to `trials.analysis`) and then synthesizes the task
   verdict (`tasks.verdict`). A sweep of `T` tasks × `N` trials therefore

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abundant-ai/oddish](https://github.com/abundant-ai/oddish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
