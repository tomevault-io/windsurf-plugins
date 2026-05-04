---
trigger: always_on
description: validates (including a concrete QA rule gatekeeper), uploads accepted
---

# AGENTS.md — Jobseek

Instructions for developer agents working on this repository.

## Project Overview

Jobseek monitors company career pages for new job postings. Companies are configured via CSV files in `apps/crawler/data/`. A Python crawler monitors boards and extracts job details. A Next.js frontend serves the data.

## Repository Structure

```
/
├── apps/
│   ├── web/                 # Next.js 15 frontend (TypeScript, Drizzle ORM, Lingui i18n)
│   └── crawler/             # Python crawler (asyncpg, httpx, structlog, redis)
│       ├── data/
│       │   ├── companies.csv    # Company registry (slug, name, website, logos)
│       │   ├── boards.csv      # Board configs (monitor + scraper per board)
│       │   └── images/          # Logo/icon staging area, uploaded to R2 by CI
│       └── src/
│           ├── core/        # Pure business logic (monitors + scrapers)
│           ├── workers/     # Worker pipeline (claim from Redis, dispatch)
│           ├── processing/  # Board/scrape processing, CPU work, R2 staging
│           ├── queries/     # SQL queries for local Postgres
│           ├── redis_queue.py # Lua-backed claim/enqueue/reschedule
│           ├── lua/         # Redis Lua scripts
│           ├── exporter.py  # CDC: local Postgres -> Supabase + Typesense
│           ├── typesense_client.py # Shared Typesense client (lazy, feature-flagged)
│           ├── sync.py      # CSV -> DB + Redis + Typesense taxonomy sync
│           ├── cli.py       # Entry point (crawler run/export/drain/sync/board/...)
│           ├── config.py    # Settings
│           └── labeller/    # Daily labelled-postings routine (Claude Code driven)
│               ├── cli.py           # labeller = src.labeller.cli:main
│               ├── normalize.py     # deterministic HTML normalizer
│               ├── blocks.py        # HTML -> block list
│               ├── validate.py      # JSON Schema + custom validators
│               ├── render.py        # Jinja task-input renderer
│               ├── sampling.py      # diverse per-company posting sample
│               ├── prepare.py       # load posting + normalize + blocks
│               ├── merge.py         # assemble subagent outputs into posting.json
│               ├── upload.py        # HuggingFace dataset push
│               ├── prompts/tasks/*.md.j2  # per-task Jinja templates
│               └── schemas/         # per-task + posting JSON Schemas
├── scripts/
│   ├── typesense-setup.py       # Create/recreate Typesense collections + aliases
│   └── typesense-backfill-local.py  # One-shot backfill from Postgres to Typesense
├── docs/                    # Architecture documentation
│   ├── 11-typesense.md      # Typesense deployment + architecture reference
│   ├── 12-typesense-benchmarks.md  # Performance benchmarks
│   ├── 14-error-review-routine.md  # Daily crawler error-review routine spec
│   └── 15-data-sampling-routine.md # Daily labelled-postings routine spec
└── .github/workflows/       # CI + agent automation
```

## Commands

Crawler (from `apps/crawler/` — see [apps/crawler/AGENTS.md](apps/crawler/AGENTS.md) for full reference):

```bash
uv sync                           # Install dependencies
uv run pytest tests/              # Run tests
uv run crawler run                # Run HTTP worker (claims from Redis simple queues)
uv run crawler run-browser        # Run browser worker (claims from Redis browser queues)
uv run crawler export             # Run CDC exporter (local Postgres -> Supabase + Typesense)
uv run crawler drain              # Run R2 description uploader
uv run crawler sync               # Sync CSVs to local Postgres + Supabase + Redis + Typesense taxonomies
uv run crawler board <slug>       # Process single board (debug)
uv run crawler backfill-typesense # Full re-index of job_posting to Typesense
uv run crawler refresh-typesense  # Refresh Typesense counts + reconcile watchlists
uv run crawler notify-indexnow    # Push changed company URLs to IndexNow (see docs/13-seo-and-indexnow.md)

# Labeller subsystem (daily gold-dataset routine — spec in docs/15-data-sampling-routine.md)
uv run labeller sample --date today --count 10 --out <path>
uv run labeller prepare <posting_id> --date today
uv run labeller render-task --task <task> --input <path> --out <path>
uv run labeller validate --kind <kind> --file <path>
uv run labeller merge --posting <id> --date <date> --out <path>
uv run labeller upload --date <date>
```

## Ops routines (Claude Code driven)

Two scheduled routines live as slash-commands + specs — orchestrated by a
Claude Code session (Opus), with specialized Sonnet subagents invoked via
the `Agent` tool. **No direct Anthropic API calls** — the data collected by
this tool feeds a future production model trained on the dataset.

- `.claude/commands/jobseek-label-daily.md` + `docs/15-data-sampling-routine.md` —
  samples diverse postings from the last 24h, labels via
  `.claude/agents/jobseek-labeller-*` subagents with tasks rendered from
  Jinja templates at `apps/crawler/src/labeller/prompts/tasks/*.md.j2`,
  validates (including a concrete QA rule gatekeeper), uploads accepted
  gold to `viktoroo/jobseek-postings-labelled` on HuggingFace.
- `docs/14-error-review-routine.md` — daily review of crawler errors on the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [colophon-group/jobseek](https://github.com/colophon-group/jobseek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
