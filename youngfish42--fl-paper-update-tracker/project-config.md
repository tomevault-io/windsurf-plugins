---
trigger: always_on
description: > This file is intended for automated coding agents (and human maintainers) who need to understand, modify, or extend the project. Keep it up-to-date whenever architecture, logic, or conventions change.
---

# Agent Guidance for FL-paper-update-tracker

> This file is intended for automated coding agents (and human maintainers) who need to understand, modify, or extend the project. Keep it up-to-date whenever architecture, logic, or conventions change.

## Project Overview

**FL-paper-update-tracker** is an automated bot that tracks new Federated Learning (FL) papers published in 40+ top-tier computer-science conferences and journals. It is a satellite project of [Awesome-FL](https://github.com/youngfish42/Awesome-FL).

### High-Level Workflow
1. GitHub Actions runs the tracker once per day (cron: `0 0 * * *`) and on every push to `main`.
2. `src/main.py` reads `config.yaml` → takes `dblp.keywords` (e.g. `[federate, FedAvg, ...]`) and `dblp.queries` (plain venue restrictions), assembles fully URL-encoded DBLP search topics, and queries the DBLP search API.
3. Extracted paper metadata is **filtered by year** (last 3 years + next 1 year) and **deduplicated by `ee` field**.
4. New papers (not yet in `cached/dblp.yaml`) are collected, formatted as Markdown, and written to the `GITHUB_ENV` variable `MSG`.
5. `scripts/convert_cache_to_md.py` regenerates `FL-Papers.md` from the updated cache.
6. If new papers exist, the action `JasonEtco/create-an-issue@v2` creates a GitHub Issue using `.github/issue-template.md`.
7. Both `cached/dblp.yaml` and `FL-Papers.md` are committed back to the repo so that subsequent runs know what has already been reported.

## Tech Stack

- **Language**: Python 3.8+
- **Core Dependencies** (see `requirements.txt`):
  - `fire` – CLI scaffolding
  - `requests` – HTTP calls to DBLP API
  - `loguru` – structured logging
  - `ezkfg` – lightweight config loader
  - `pyyaml` – cache read/write
- **CI/CD**: GitHub Actions (Ubuntu runner)
- **License**: Apache 2.0

## Directory Structure

```
.
├── .github/
│   ├── workflows/watch.yml          # GitHub Actions workflow definition
│   └── issue-template.md            # Nunjucks template for auto-created issues
├── cached/
│   └── dblp.yaml                    # Persistent cache of already-reported papers
├── scripts/
│   ├── convert_cache_to_md.py       # Converts cache to structured Markdown (domain-specific maps)
│   ├── fetch_abstracts.py           # Backfill/refresh paper abstracts via external APIs
│   ├── fetch_dois.py                # Backfill missing DOIs via DBLP / Crossref / Semantic Scholar
│   ├── dedup_cache_by_title.py      # Deduplicate cache entries by title
│   └── dedup_cache_global.py        # Global cross-topic deduplication for the cache
├── src/
│   ├── main.py                      # Entry point: assembles topics from keyword+queries, orchestrates API calls
│   └── utils.py                     # Helper functions: API call, parsing, formatting, dedup
├── config.yaml                      # keyword, plain queries (venues), and mail targets
├── FL-Papers.md                     # Structured Markdown output of all tracked papers
├── requirements.txt                 # Python dependencies
├── README.md                        # Human-facing documentation (EN + CN)
├── TECHNICAL.md                     # Deployment and configuration guide
└── AGENTS.md                        # This file
```

## Key Logic & Conventions

### 1. Year Filtering
- Location: `src/main.py` (calls `filter_items_by_year` in `src/utils.py`)
- Rule: Only papers whose `year` falls in `[current_year - 3, current_year + 1]` are kept.
- Example: In 2026, valid years are 2023–2027.
- **Agent Note**: If you change this window, update both the implementation **and** this document.

### 2. Deduplication
- Location: `src/utils.py` + `src/main.py`
- **Three-stage dedup**:
  1. `deduplicate_items_by_ee` (per-topic) — Within a single DBLP query result, papers with identical `ee` are deduplicated. Rationale: DBLP sometimes returns multiple records for the same paper with minor author-name differences (e.g., `Ming Hu 0003` vs `Ming Hu`).
  2. `deduplicate_items_by_title` (per-topic) — Within a single query result, papers with identical `title` are also deduplicated. Rationale: DBLP may list the same paper multiple times under different `ee` URLs (e.g., preprint vs. proceedings version).
  3. **Global cross-topic dedup** (in `main.py`) — A paper that has already been cached under any topic is skipped when processing subsequent topics. Rationale: DBLP search API can return the same paper for multiple venue queries (e.g., a keyword match may cross venue boundaries), so a global `seen_ee`/`seen_title` set prevents the same paper from being stored under multiple topic keys.
- **Agent Note**: Do **not** switch back to full-dict comparison (`item not in cached_items`) unless you also normalize author names.

### 3. Cache Format (`cached/dblp.yaml`)
- Top-level keys: URL-encoded DBLP search topics (e.g., `federate%20venue%3ADAC%3A:` or `FedAvg%20venue%3AICML%3A:`).
- Each key maps to a list of paper dicts with fields: `author`, `title`, `venue`, `year`, `type`, `access`, `key`, `doi`, `ee`, `url`, `abstract`, `abstract_cn`, `related_code`.
  - `abstract`  may be empty for legacy entries; use `scripts/fetch_abstracts.py` to backfill it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [youngfish42/FL-paper-update-tracker](https://github.com/youngfish42/FL-paper-update-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
