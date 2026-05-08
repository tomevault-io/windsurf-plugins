---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## What This Is

**Social Media AI** — a platform for Aysun's social media agency (Sun x ca) that generates viral Instagram Reels content. It scrapes competitors, analyzes viral videos with AI, generates content strategies, and writes complete video scripts — all optimized for each client's unique voice and brand.

---

## How to Run

```bash
npm install
npm run dev
# Open http://localhost:3000
```

**Required environment variables** (in `.env` at project root):
- `APIFY_API_TOKEN` — Apify Instagram scraper
- `GEMINI_API_KEY` — Google Gemini video analysis
- `ANTHROPIC_API_KEY` — Claude concept generation
- `BRAVE_API_KEY` — Brave Search API for live trend research
- `JOB_SECRET` — Auth token for background research jobs
- `GOOGLE_SERVICE_ACCOUNT_KEY` — Google Drive integration (base64 JSON)

---

## Tech Stack

- **Next.js 16** (App Router) + **TypeScript**
- **Tailwind CSS** + **shadcn/ui** components
- **Supabase** for data storage (migrated from CSV)
- **Apify** — Instagram scraping
- **Google Gemini 2.0 Flash** — Video analysis (upload + multimodal)
- **Claude Sonnet** — Script generation, strategy, concept adaptation

---

## How The System Works

### Video Analysis Pipeline

1. **Input** — Select a config and parameters (max videos, top-K, days lookback) via the Run page
2. **Load Config** — Retrieve analysis prompt, new concepts prompt, and creator list
3. **Scrape** — For each competitor creator, scrape recent Instagram Reels via Apify
4. **Filter & Rank** — Filter by date, sort by views, take top-K most viral
5. **Analyze** — Download video, upload to Gemini, analyze (extracts Concept, Hook, Retention, Reward, Script)
6. **Generate** — Send analysis + brand context to Claude for adapted video concepts
7. **Save** — Results viewable in the Videos page with thumbnails

### Strategy Generation Pipeline — Multi-Step SSE Pipeline

1. **Load Context** — Client profile, audit, performance, competitors, voice profile (no data truncation)
2. **Data Analysis & Goal** — Single Claude call analyzing all data, determining reach/trust/revenue goal
3. **Strategy Creation** — Single Claude call creating 3-5 content pillars with video ideas + weekly plan
4. **Strategy Review** — Single Claude call checking consistency, voice-format match, subtopic quality

Key endpoint: `POST /api/configs/[id]/generate-strategy` (SSE stream)

### Weekly Ideas Pipeline — One-Shot Opus

Note: despite the endpoint name `generate-week-scripts`, this flow produces **ideas, not scripts**. Scripts are generated on-demand from an idea (see next section).

1. **Lock** — `acquirePipelineLock()` prevents parallel week runs per client (returns 409 on collision)
2. **Parallel Context Load** — `loadPipelineContext()` + `loadVoiceProfiles()`:
   - Client/brand/audit/performance/competitor/cross-niche context
   - Week-seeded schedule (same configId + same ISO week → same day/pillar/type/format rotation, different week → different offset)
   - Winner anchors (own top videos + competitor winners for core/variant posts)
   - Recent scripts + hook-pattern history (to avoid recycling)
   - Audit-preferred duration as hard ceiling
   - Voice profile + script structure + voice onboarding block (all cached in DB, regenerated on miss)
3. **Research** — `runResearch()`: Brave deep search (15-20 queries across 9 categories, week-RNG rotates angles) → Sonnet synthesizes 6-12 real trends via `TREND_RESEARCH_TOOL` (plus high-confidence client learnings from Supabase)
4. **One-Shot Idea Generation** — single **Opus** call (`claude-opus-4-7`) with the `weekly-ideas` agent + `WEEKLY_IDEAS_TOOL`. Sees full context and returns N coherent ideas (one per active day) with `title`, `angle`, `hookDirection`, `keyPoints`, `whyNow`, `emotion`, plus a `weekReasoning`
5. **Return Inline, Don't Persist** — ideas stream back to the UI. The user picks which to develop into full scripts via the Content Agent chat or single-script generator
6. **Fire-and-Forget** — triggers `/api/jobs/research-cycle` to refresh snapshots/learnings for the next run

Entry point: `src/lib/pipelines/weekly-oneshot.ts` (`generateWeekIdeas`)
Shared setup: `src/lib/pipelines/weekly-steps.ts` (`loadPipelineContext`, `loadVoiceProfiles`, `runResearch`)
Route orchestrator: ~95 lines — `POST /api/configs/[id]/generate-week-scripts` (SSE stream)
Voice profile: `POST /api/configs/[id]/generate-voice-profile`

### Single Script Generation — From Idea to Full Script

Scripts are always generated **per-idea**, not in batch. Two paths:

**Via Content Agent chat** (`POST /api/chat`, SSE): the agent calls its `generate_script` tool which runs the Script Writer (`script-writer.md`) → regex quality check → Reviewer (`script-reviewer.md`) only if issues found. Always returns short (30-40s) + long (60+s) versions.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sunxcamarketing/social-media-ai](https://github.com/sunxcamarketing/social-media-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
