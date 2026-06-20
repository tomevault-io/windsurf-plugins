---
trigger: always_on
description: Full Instagram content engine powered by the Socioverse API. Runs competitor intelligence, AI ideation, content production (carousels + captions), scheduling, DM automation, lead management, and weekly reporting - all end-to-end. Use when the user wants to run the content pipeline, generate ideas, schedule posts, manage automations, handle leads, send DMs, analyze competitors, manage the content bank, use IdeaPad, generate reports, create carousels, write captions, or do anything with Socioverse
---


# Socioverse - Industry-Ready Content Engine

Full-lifecycle Instagram content automation: intelligence → ideation → production → scheduling → engagement → reporting.

**Base URL:** `https://api.socioverse.io`
**Auth:** `Authorization: Bearer $SOCIOVERSE_API_KEY`
**Rate limits:** 60 req/min, 1000 req/day
**Key format:** `sk_live_*` (get from dashboard → Settings → API Keys)

---

## Quick Start

```bash
# Verify setup
/socioverse status

# Full weekly content pipeline
/socioverse pipeline

# Just generate ideas
/socioverse ideate --topic "founder burnout"

# Check what user approved in IdeaPad
/socioverse review

# Produce approved ideas into content
/socioverse produce

# Schedule everything
/socioverse schedule
```

---

## Modes

| Mode | Command | Description |
|------|---------|-------------|
| Status | `/socioverse status` | Health check, usage stats, connected accounts |
| Intel | `/socioverse intel` | Competitor scrape + analysis + intelligence brief |
| Ideate | `/socioverse ideate` | Generate ideas → push to IdeaPad |
| Review | `/socioverse review` | Pull IdeaPad, show approved/rejected/pending, iterate |
| Produce | `/socioverse produce` | Accepted ideas → scripts/carousels → Content Bank |
| Schedule | `/socioverse schedule` | Content Bank → scheduled Instagram posts |
| Pipeline | `/socioverse pipeline` | Full orchestration (intel→ideate→pause→produce→schedule) |
| Automate | `/socioverse automate` | DM automations, sequences, lead magnets |
| Leads | `/socioverse leads` | Lead CRUD, messaging, profile lookup |
| Report | `/socioverse report` | Weekly reports, insights, analytics dashboard |

---

## Environment Setup

The skill reads `SOCIOVERSE_API_KEY` from environment. For public users:

```bash
# In your project .env or shell
export SOCIOVERSE_API_KEY=sk_live_your_key_here
```

All API calls use:
```bash
curl -s -X {METHOD} "https://api.socioverse.io{ENDPOINT}" \
  -H "Authorization: Bearer $SOCIOVERSE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{BODY}'
```

---

## Mode 1: Status

**Command:** `/socioverse status`

Performs a full account health check.

### Steps:
1. `GET /v1/health` - verify key validity
2. `GET /v1/usage` - 30-day usage stats
3. `GET /v1/keys/verify` - active scopes
4. `GET /v1/instagram/accounts` - connected accounts
5. `GET /v1/competitors` - tracked competitors count
6. `GET /v1/automations` - active automations count
7. `GET /v1/leads?limit=1` - total leads (from pagination meta)

### Output:
Present a dashboard table:
```
┌─────────────────────────────────────────┐
│ SOCIOVERSE STATUS                       │
├─────────────────────────────────────────┤
│ API Key:      sk_live_ab12...  ✓ Active │
│ Scopes:       21/21 enabled            │
│ Plan:         Growth                    │
│ Rate:         58/60 remaining          │
│ Daily:        847/1000 remaining       │
├─────────────────────────────────────────┤
│ IG Accounts:  2 connected              │
│ Competitors:  5 tracked                │
│ Automations:  3 active                 │
│ Leads:        127 total                │
│ Posts queued:  12 scheduled            │
└─────────────────────────────────────────┘
```

---

## Mode 2: Intel

**Command:** `/socioverse intel [--analyze-count N] [--competitors ID1,ID2]`

Builds an intelligence brief from competitor data and knowledge base.

### Parameters:
| Flag | Default | Description |
|------|---------|-------------|
| `--analyze-count` | 6 | Number of top posts to deep-analyze |
| `--competitors` | all | Specific competitor IDs (comma-sep) |
| `--skip-scrape` | false | Use existing scraped posts only |

### Steps:

#### Step 1: Pull Knowledge Context
```
GET /v1/knowledge
```
Read all knowledge documents to understand the user's brand voice, audience, and positioning. This context informs all downstream generation.

#### Step 2: Scrape Competitors (async)
```
POST /v1/competitors/scrape
{ "batch_size": 12, "post_types": ["reel", "carousel", "image"] }
```
If `--competitors` specified, add `"competitor_ids": [...]`.

Response includes `pipeline_run_id`. The scrape runs asynchronously - the response returns BEFORE the posts are queryable. Do not try to read scraped-posts immediately.

#### Step 3: Wait for the scrape run to finish, then read posts

Poll the pipeline run until terminal:
```
GET /v1/content-studio/pipeline-runs/{pipeline_run_id}
```
Response includes:
- `status`: `running | completed | failed | cancelled`
- `is_terminal`: `true` when status is completed/failed/cancelled
- `items_processed`, `items_failed`: scrape progress
- `posts_attached`: count of scraped_posts created by this run

Poll every 3-5s until `is_terminal: true`. Typical scrape runs complete in 30-90s depending on batch size.

Then fetch the actual posts:
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnshulRastogi20/socioverse-skill](https://github.com/AnshulRastogi20/socioverse-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
