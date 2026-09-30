---
trigger: always_on
description: Instructions for AI coding agents (Codex, Copilot, Claude, Cursor, …) working in this repo.
---

# AGENTS.md

Instructions for AI coding agents (Codex, Copilot, Claude, Cursor, …) working in this repo.
Humans should read [CONTRIBUTING.md](CONTRIBUTING.md). Operators should read
[docs/operations.md](docs/operations.md).

## What this is

New Grad Jobs is an automated board of entry-level jobs in every field (software, data,
engineering, finance, marketing, sales, healthcare, …) in the US, Canada and India.
A Python scraper pulls public ATS APIs (Greenhouse, Lever, Ashby, Workday) plus JobSpy
(Indeed) about every 30 minutes in GitHub Actions. It filters for new-grad roles and
publishes static JSON/RSS. A Vite + React site at <https://jobs.riteshrana.engineer> is
built from that data and deployed to GitHub Pages. There is no server and no database.

## Repo map

```
config.yml                 companies per ATS, filter signals, source knobs (validated by scripts/validate_config.py)
scripts/update_jobs.py     CLI entrypoint → ngj.pipeline.main
scripts/ngj/               the scraper package
  pipeline.py              fetch → dedup → filter → enrich → URL gate → collapse guard → publish
  settings.py              frozen Settings built from config.yml + env (NGJ_OUTPUT_DIR, NGJ_SITE_URL)
  models.py                SourceResult / SourceError (per-company errors travel with the jobs)
  registry.py              which sources are enabled (shared by pipeline, health.json, validate_config)
  http.py                  shared requests session, per-domain concurrency limits, 403 cooldown hooks
  sources/                 one adapter per source; each returns a SourceResult
  filters.py locations.py  inclusion gate: hard rules (exclusions, new-grad/track signals) and soft rules
                           (intern/co-op, level III+, recency, US/CA/IN) whose failures become near misses
  dedup.py                 same-posting (job_id) + cross-source dedup
  taxonomy.py enrich.py    CATEGORY_PATTERNS, company tiers, sponsorship/closed flags
  outputs/                 jobs_json, rss, health, market_history, previous (last published run)
scripts/contracts.py       jobs.json schema (1.1), canonical_url, compute_job_id
scripts/publish.py         jobs-index.json + descriptions/<0-f>.json shards
scripts/quality.py         cross-artifact integrity checks (run by scripts/check_integrity.py)
scripts/url_safety.py      publish-time URL gate (public http(s) only)
scripts/sync_readme_*.py   rewrite README COUNT markers / CATEGORY-LISTINGS / COMPANY-LISTINGS blocks
tests/                     pytest; network blocked by tests/conftest.py
site/                      Vite + React 18 app (src/components, src/lib, src/hooks, src/data); tabs: hiring,
                           contributors, explore (every posting the run saw, viewer-defined signals over corpus-index.json)
site/scripts/seo/          Vite plugin: CSP, /job/<job_id>/ pages with JobPosting JSON-LD, /jobs/… landing pages (landing.mjs),
                           /guides/ from site/content/guides/*.md (guides.mjs), /about/ from health.json (about.mjs), sitemap, robots, prerender
site/content/guides/       evergreen guides (Markdown with front matter: title, description, updated)
data/market-history.json   daily snapshots (committed by CI, 90-day retention)
docs/                      architecture.md, operations.md, adr/, removed-companies.md
```

## Setup

```bash
make setup                 # .venv + hash-locked runtime/dev deps + pre-commit hook (Python 3.11+)
make install               # re-install deps into an existing .venv
cd site && npm ci          # Node >= 20.19 (CI uses 22)
```

## Commands

```bash
make test                  # pytest; coverage floor 75% (pyproject.toml)
make lint                  # ruff + pre-commit --all-files
make format                # ruff --fix
make typecheck             # mypy (CI job is non-blocking for now)
.venv/bin/python scripts/validate_config.py          # after any config.yml edit
NGJ_OUTPUT_DIR=/tmp/ngj make run                     # real scrape (network, minutes)
.venv/bin/python scripts/check_integrity.py /tmp/ngj # validate that output

cd site
npm run fetch-data         # download live jobs/health/feed/descriptions into public/ (gitignored)
npm run dev                # dev server
npm test                   # vitest (src/lib coverage threshold 80% lines)
npm run lint               # eslint
npm run build              # dist/ incl. SEO pages; tolerates missing data
npm run build:fixtures     # build against site/test/fixtures
```

After `make run`, restore the two files that a local scrape rewrites:
`git checkout -- README.md data/market-history.json`.

## Architecture in brief

- **Pipeline** (`ngj.pipeline.run`): `plan_sources` binds one fetcher per enabled source
  (from `ngj.registry`) to `Settings`, and `fetch_all_sources` runs them concurrently.
  Then `deduplicate_jobs` → `filter_jobs` → `enrich_jobs` → `filter_safe_jobs` →
  `generate_jobs_json` → partial-collapse guard → `_publish` (history, jobs artifacts,
  feed, health, README sync). The run exits 1 on any write failure or a guard trip.
- **Errors are data:** adapters never raise for a single company. They return
  `SourceResult(jobs, errors=(SourceError(company, source, kind, status, message), ...))`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ambicuity/New-Grad-Jobs](https://github.com/ambicuity/New-Grad-Jobs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
