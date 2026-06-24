---
trigger: always_on
description: Guidance for future Codex sessions working in this repo.
---

# AGENTS.md

Guidance for future Codex sessions working in this repo.

## Overview

`pupila` is a **config-driven, forkable** daily job aggregator. It fetches listings from 13 public sources (3 ATS APIs — Ashby, Greenhouse, Lever — plus RSS feeds, JSON job boards, Hacker News, HTML scrapers, an Aave Next.js scraper, and `ashby-private` — a config-driven fetcher for orgs hosted on Ashby with the public posting-API disabled), normalizes them, applies hard exclusion filters, computes a per-job `fitScore`, deduplicates, and writes `data/jobs.json`, an RSS feed at `data/feed.xml`, and an auto-regenerated `JOBS.md` table. The hand-written `README.md` is the project doc and is **not** rewritten by the pipeline. No external services. No DB. Output lives in this repo.

The repo ships a **neutral template** in `config/profile.json`. After onboarding (CV upload → brief generation), `/api/profile-generate` shells out to the local LLM CLI to fill in the personal keyword lists + weights based on the brief. Re-runnable from Settings → Scoring profile → Regenerate. `config/slugs.json` ships with the full ~50-company tier-S list (all public ATS URLs — non-personal data, edit by hand to add/remove companies).

**First-run UX**: a forker generates their `config/candidate-brief.md` by running `pnpm run setup-brief --file ~/cv.pdf` (or via the UI's Profile tab → drop a PDF/DOCX/MD CV). That CLI shells out to whichever local LLM CLI is installed (`Codex`, `codex`, `gemini`, `opencode` — auto-detected, override via `PUPILA_LLM=<provider>`).

## Stack

- Node 22 LTS, ESM, TypeScript 5.9 (NodeNext)
- Biome 2.4 (lint + format, single config in `biome.json`)
- pnpm 10
- Vitest 3 (tests in `tests/`, run via `pnpm test` — 120 cases)
- simple-git-hooks (pre-commit `lint && typecheck`)
- Single runtime dep: `fast-xml-parser`. Native `fetch` only.

## Run locally

```bash
pnpm install                # also installs the pre-commit hook
pnpm run dev                # tsx, no build step (this is what the launchd/cron aggregate agent runs)
pnpm start                  # built output: requires pnpm run build first
pnpm run typecheck          # tsc --noEmit on src/, then on src/+tests/ via tsconfig.test.json
pnpm run lint               # biome check
pnpm run lint:fix            # biome check --write
pnpm test                   # vitest run (120 unit tests)
pnpm run test:watch         # vitest watch mode
pnpm run ui                 # local-only UI: Vite dev server on http://127.0.0.1:5173, reads data/jobs.json
pnpm run ai-review          # local-only: shells out to `Codex -p` to write data/ai-reviews.json
pnpm run ai-review --top=50 # raise the per-run cap (default 20 highest-fitScore unreviewed)
pnpm run ai-review --force  # re-review entries that already exist
pnpm run ai-review --ids=a,b # review specific job ids only
pnpm run setup-brief --file ~/cv.pdf  # generate config/candidate-brief.md from a CV (PDF/DOCX/MD/TXT)
pnpm run daily              # convenience: pnpm run dev && pnpm run ai-review (morning routine)
```

The pipeline writes to `data/jobs.json` (slim — `body` field stripped), `data/feed.xml` (RSS 2.0 of new jobs), `JOBS.md`, optionally `data/archive/<YYYY-MM>.json` on day 1 of the month, and per-source raw dumps in `data/raw/<source>-<YYYY-MM-DD>.json` (gitignored). `README.md` is hand-maintained — never overwrite it from code.

Pre-commit runs `lint && typecheck` automatically. Bypass with `SKIP_SIMPLE_GIT_HOOKS=1 git commit ...` for emergency commits.

## Repo layout

```
src/
  index.ts          # orchestrator
  types.ts          # Job, Source, Category, ApplicationStatus, AppliedEntry, FetcherResult, Raw* shapes
  utils.ts          # fetchWithTimeout (1-retry), isSafeUrl, sha1, stripHtml, readJsonOrNull, ...
  rss.ts            # shared fast-xml-parser wrapper
  normalize.ts      # one normalize<Source> per source -> Job (uses withSalary() for parsed salary fields)
  salary.ts         # parseSalary(): raw string -> { min, max, currency } (annual integer, ISO code)
  filters.ts        # createFilters(profile) factory + applyFilters default. Hard excludes + scoring + category + _signals.
  applied.ts        # loads config/applied.json, attaches AppliedEntry to Job by URL hash
  dedup.ts          # 2-pass dedup, priority-aware tiebreak
  render.ts         # JOBS.md markdown (applied, ✨ new, 🗑 removed, 🚨 source-health, status/salary in title)
  feed.ts           # RSS 2.0 generator -> data/feed.xml (top 50 ✨ new jobs)
  setup-brief.ts    # CLI: parse CV (pdf/docx/md/txt) → LLM CLI → write config/candidate-brief.md
  lib/
    llm.ts            # detectLlmCli + runLlm — provider-agnostic shell-out (Codex/codex/gemini/opencode)
    cv-parser.ts      # parseCvBuffer + parseCvFile (pdfjs-dist for PDF, mammoth for DOCX, raw text otherwise)
    brief-template.ts # readBriefBody / writeBriefBody — preserve preamble + markers in candidate-brief.md
  fetchers/
    _shared.ts                                              # fetchMultiSlug orchestration helper
    ashby.ts            greenhouse.ts       lever.ts        # ATS APIs (public, multi-slug)
    ashby-private.ts                                        # multi-slug GraphQL for hidden Ashby orgs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FranRom/pupila](https://github.com/FranRom/pupila) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
