---
trigger: always_on
description: This file is the **schema**: it tells you (the LLM) how to operate this workspace. Everything else is either a raw source, a wiki page, or a meta file. Read this on every session start. Active workstreams + open decisions live in `ROADMAP.md`, not here.
---

# SEO / GEO / B&M Business Research Workspace — Schema

This file is the **schema**: it tells you (the LLM) how to operate this workspace. Everything else is either a raw source, a wiki page, or a meta file. Read this on every session start. Active workstreams + open decisions live in `ROADMAP.md`, not here.

## Purpose

Local knowledge hub for **SEO, local search (geographic SEO), Generative Engine Optimization (GEO/AEO), web design, and social media** — scoped to two verticals:

1. **Brick-and-mortar operators** (single- or multi-location) who need to rank in local search, be cited correctly by AI engines, and run their owned + earned digital surfaces. The wiki uses a barbershop running example throughout because that's the seed domain it was built from, but the principles, tools, and playbooks generalize to any local service business — restaurants, dental clinics, auto shops, salons, gyms, retail.

2. **Creator-marketing operators** (subscription content platforms like OnlyFans, Patreon, Buy Me a Coffee) who need to grow an audience, convert free followers to paid subscribers, retain existing subscribers, and drive external traffic from social platforms (Twitter/X, Reddit, TikTok, Instagram) to their subscription page. The wiki uses a friends OnlyFans creator as a running example, but the principles generalize to any image-based subscription content creator.

The wiki is a librarian that **manages, curates, and applies** that knowledge:

The wiki is a librarian that **manages, curates, and applies** that knowledge:

- **Manage** — inventory raw sources (best-practice articles, Google Business Profile docs, schema specs, marketing case studies, SEO-tool docs, social-media playbooks); track what's been read, extracted, and applied
- **Curate** — pull relevant fragments out of raw sources; structure them as interlinked wiki pages on platforms (GBP, Yelp, Instagram, etc.), tools, concepts (local pack, citations, schema, reviews), and the operator's two physical shops
- **Apply** — route findings to a real workflow:
  - **claude.ai / Claude Desktop** — context for writing review responses, drafting website copy, generating social-media captions, brainstorming local content
  - **Direct hands-on use** — paste a brief into the operator's website CMS, GBP dashboard, Instagram, or email reply

This is a laptop-only workspace. No remote servers, no team distribution. Everything lives on this MacBook.

## Two meanings of "GEO" — both apply

This domain has a vocabulary collision. **Both meanings are in scope** for this wiki:

- **Geographic SEO** — ranking for location-bound queries ("barbershop near me", "fade haircut [city] [st]"). The classical local-search discipline: Google Business Profile, NAP consistency, citations, the local pack / map pack, reviews, geo-targeted on-page content.
- **Generative Engine Optimization (GEO) / Answer Engine Optimization (AEO)** — getting cited and accurately represented in answers from ChatGPT, Claude, Perplexity, Google AI Overviews, Gemini. A 2024+ discipline; rapidly mattering for "best [category] in [city]" type queries that increasingly resolve in AI surfaces before users click anywhere.

When the wiki refers to "GEO" without qualifier, default to **geographic SEO** unless context makes the AI-engine variant clear. Tag pages with `geo-search` vs `geo-aeo` to disambiguate.

## Architecture — three layers

1. **Raw sources** — immutable. You read them, never modify them. Live locally in `raw-sources/` (gitignored — articles, screenshots, PDFs, repo snapshots).
   - Articles, blog posts, video transcripts saved as `.md`
   - PDFs (e-books, vendor whitepapers, conference talks)
   - GitHub repos (cloned snapshots of FOSS local-SEO tools, schema generators, review-management scripts)
   - Screenshots of competitor GBP listings, search SERPs, Instagram profiles
   - **Drop pattern**: drop new sources into `research to be indexed/` (transient drop zone). Ingest pipeline reads + synthesizes, then move to `raw-sources/`.

2. **The wiki** — LLM-written, human-read. Lives in `wiki/`. Structured pages on platforms, tools, concepts, markets, and the operator's shops.

3. **The schema** — this file.

Staging/output lives outside the wiki:
- `briefs/` — one-off deliverables (gitignored): a review-response template pack, an Instagram content calendar, a website-copy revision, a competitor analysis report
- `research to be indexed/` — transient drop zone for new raw sources (gitignored)
- `LESSONS.md` — meta-lessons about *how we work* (distinct from `wiki/log.md`)
- `hot.md` — ephemeral session-state cache (gitignored)
- `ROADMAP.md` — active workstreams + open decisions (tracked)

## Folder layout

```
SEO-GEO-B-M-Wiki/                   # repo root (folder name when cloned from GitHub)
  CLAUDE.md                         # this file — the schema
  LESSONS.md                        # meta-lessons (how we work)
  ROADMAP.md                        # active workstreams + decisions + done log
  hot.md                            # session-state cache (gitignored)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cemini23/SEO-GEO-B-M-Wiki](https://github.com/cemini23/SEO-GEO-B-M-Wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
