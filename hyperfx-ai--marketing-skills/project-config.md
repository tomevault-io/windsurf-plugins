---
trigger: always_on
description: Guidance for AI coding agents (Codex, Cursor, Claude Code, etc.) working in this repository.
---

# AGENTS.md

Guidance for AI coding agents (Codex, Cursor, Claude Code, etc.) working in this repository.

## What this repo is

A public collection of marketing-domain Agent Skills that wrap the **Hyper MCP** tool surface. Each skill in [`skills/`](./skills) teaches an agent how to drive Hyper's MCP-exposed tools (Google Ads, Meta, Amazon, TikTok, Pinterest, LinkedIn, HyperSEO, Apify scrapers, image / video generation, Gmail, Apollo, Klaviyo, Resend, Beehiiv, GA4, GTM, Search Console, BigQuery, etc.) for a specific marketing job.

Skills are NOT tools. They are markdown instructions plus optional reference files. The actual tool calls come from the connected Hyper MCP at [https://app.hyperfx.ai/mcp](https://app.hyperfx.ai/mcp). The MCP descriptor for each tool is the only source of truth for tool names, arg names, and arg types — never trust skill content over the live MCP catalog when they disagree.

## Available skills

### Paid ads

| Skill | What it does |
| --- | --- |
| [`google-ads`](./skills/google-ads) | Plan and create new Google Ads campaigns (Search, Display, Performance Max) using the blueprint flow. |
| [`meta-ads`](./skills/meta-ads) | Plan and create Meta (Facebook/Instagram) ad campaigns with Advantage+ as the default. |
| [`meta-ads-library`](./skills/meta-ads-library) | Search the Meta Ads Library for competitor ads, optionally enrich with page contact info, and surface structured ad-intelligence summaries inline in chat. |
| [`amazon-ads`](./skills/amazon-ads) | Plan and create Amazon Sponsored Products campaigns with strategic targeting and budget rules. |
| [`tiktok-ads`](./skills/tiktok-ads) | Plan and create TikTok Ads (Traffic, Reach, Conversions, App Promotion, Video View) with strict objective-specific parameter validation. |
| [`pinterest-ads`](./skills/pinterest-ads) | Plan and create Pinterest Ads with microcurrency budgeting, CBO rules, and conversion-tag handling. |

### Organic social publishing

| Skill | What it does |
| --- | --- |
| [`tiktok`](./skills/tiktok) | Publish organic TikTok videos, photos, and carousels through the compliance posting form. |
| [`instagram`](./skills/instagram) | Publish photos / Reels / Stories / carousels, moderate comments and mentions, send DMs, and pull insights from the Instagram API with Instagram Login. |
| [`linkedin`](./skills/linkedin) | Publish LinkedIn text, article, document, organization (company page), and AI text-to-carousel posts. |

### Research

| Skill | What it does |
| --- | --- |
| [`seo-research`](./skills/seo-research) | Data-driven SEO research via the HyperSEO toolkit — keyword research, SERP and AI Overview analysis, competitor benchmarking, content planning, AI search visibility tracking (ChatGPT / Claude / Perplexity), backlink trends, and full site audits. Ships with five reference workflows under `references/`. |
| [`competitor-intel`](./skills/competitor-intel) | End-to-end competitor research and monitoring — pick the set, scrape every public surface (Firecrawl for site/blog/pricing, HyperSEO for rank/backlinks/intersection, Apify scrapers for Instagram / TikTok / LinkedIn / Twitter / Reddit / Google search / Google Trends), diff against last run, and synthesize battle cards / weekly digests / comparison-page input / board-prep updates. Ships with two references under `references/` (per-source playbook, brief templates). |

### Creative output

| Skill | What it does |
| --- | --- |
| [`ad-creative-generation`](./skills/ad-creative-generation) | End-to-end ad creative pipeline — brand extraction (Firecrawl) → ad copy variants → on-brand images for Google RSAs, Display, Performance Max, Meta feed / story / carousel, and other paid placements. Ships with four reference workflows under `references/`. |
| [`image-generation`](./skills/image-generation) | Tool-selection guide for the bundled image models — OpenAI (`gpt-image-2`), Nano Banana (Gemini 3 Pro / 2.5 Flash), Seedream (ByteDance via fal.ai) — for ad creatives, branded compositions, photoreal product shots, image-to-image edits, and high-resolution output. |
| [`video-generation`](./skills/video-generation) | End-to-end AI video — text-to-video / image-to-video (Sora, Veo, Seedance), scene chaining, video analysis, transcription, subtitles, TikTok / karaoke captions, voiceover (TTS), audio mixing, clipping, stitching, and text overlays. |

### Outbound & lifecycle

| Skill | What it does |
| --- | --- |
| [`cold-email-outreach`](./skills/cold-email-outreach) | End-to-end B2B cold outreach — Apollo prospect search and email enrichment, Firecrawl + LinkedIn-scraper personalization signals, Gmail drafting / sending / threading, multi-touch follow-up cadences, and reply routing via Gmail labels. Ships with three reference workflows under `references/` (frameworks, follow-ups, deliverability). |
| [`email-lifecycle`](./skills/email-lifecycle) | Welcome / nurture / re-engagement / win-back / abandoned-cart sequences across Klaviyo (ecom), Resend (SaaS / dev tools), Beehiiv (newsletters), and Gmail (small lists). Provider-selection guidance, audience setup, sequence build, launch, and measurement. Ships with two references under `references/` (cross-provider sequence patterns and per-provider tool-call mechanics). |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyperfx-ai/marketing-skills](https://github.com/hyperfx-ai/marketing-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
