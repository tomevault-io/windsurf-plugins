---
trigger: always_on
description: Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.
---

# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

# CLAUDE.md — job-scrapper

Project root: `/home/$USER/www/job-scrapper`

> Last updated: 2026-05-10
> Update this file when: model status changes, new API endpoints, new hard rules, pipeline prompt changes.

---

## What this project does

Job scraper + fit analyzer + local viewer. It crawls job boards via ATS provider APIs, scores listings against a candidate profile using LLMs, and surfaces results through a local web UI with filtering, analysis pipelines, and a Discord notification system.

---

## Architecture overview

```
job-scrapper/
├── crawler/          # Provider-specific scrapers
├── matcher/          # Python — LLM scoring pipelines
│   ├── job_post_parser.py
│   ├── job_fit_analyzer.py     # Quick / Maverick pipeline
│   ├── ensemble_runner.py      # Full / Ensemble pipeline
│   ├── benchmark.py
│   ├── benchmark_ensemble.py
│   └── career-ops/             # User-owned profile files — NEVER TOUCH
│       ├── _profile.md
│       ├── profile.yml
│       ├── cv.md
│       └── portals.yml
└── viewer/           # Node/Express + TypeScript
    ├── src/
    │   ├── server.ts           # Express API + SQLite
    │   └── lib/
    │       ├── types.ts
    │       ├── queue.ts
    │       └── config.ts
    └── public/
        ├── index.html          # Active UI (index.old.html = retired Bootstrap UI)
        ├── feed.js             # All UI logic
        ├── feed.css            # All styles
        └── saved-searches.json # User-editable search presets
```

---

## Docker policy — MANDATORY

**Never run npm / npx / python directly. Always use Docker.**

| Change type | Command |
|---|---|
| `server.ts` or any TypeScript | `docker compose build viewer && docker compose up -d viewer` |
| Static files (`feed.js`, `feed.css`, `index.html`) | `docker compose build viewer && docker compose up -d viewer` |
| Python matcher changes | `docker compose build matcher` |

Rebuild automatically when files change — do not ask for permission first.

---

## Environment variables (docker-compose.yml)

| Var | Purpose |
|---|---|
| `NVIDIA_API_KEY` | Required for all LLM calls |
| `NVIDIA_MODEL` | Default scorer — overrides Python default |
| `NVIDIA_ENSEMBLE_SCORERS` | Comma-separated scorer model list |
| `CATALOG_DB` | `/app/state/catalog.sqlite` |
| `CAREER_OPS_DIR` | Profile directory inside matcher, default `career-ops` |
| `SCORE_NOTIFY_MIN_SCORE` | Min score threshold for Discord notifications and To-Apply bucket |

---

## API endpoints (server.ts)

| Method | Path | Notes |
|---|---|---|
| GET | `/api/jobs` | Paginated job list. Filters: `title`, `location`, `company`, `sources`, `days`, `page`. `days` filters by `first_seen_at` |
| GET | `/api/job` | Single job + analysis. Params: `provider`, `source_key`, `job_id` |
| GET | `/api/job-parsed` | Live parse via `job_post_parser`. Query params only — Workday `source_key` contains `/` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bracketouverte/job-crawler](https://github.com/bracketouverte/job-crawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
