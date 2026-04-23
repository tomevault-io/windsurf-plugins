---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md — tavr-digest

This file provides guidance to Claude Code when working with this repository.

## Project Overview

**The Valve Wire** is a production daily automation pipeline that aggregates structural heart disease news — covering transcatheter AND surgical approaches to aortic (TAVR/TAVI), mitral (MitraClip, PASCAL, TMVR), and tricuspid (TriClip, TTVR) valves — from academic, regulatory, financial, and social sources. It uses Claude AI to synthesize a daily newsletter, publishes to a companion website, generates a weekly podcast, and is monitored via a Telegram bot.

**Goal:** Commercialization — paid subscriptions, sponsorships, and evolving into a platform that can spin up automated digests for any niche.

**Website:** The Valve Wire website lives inside this repo at `site/` (Next.js, deployed on Vercel from `site/` root directory). The original standalone repo was renamed to [medweb-template](https://github.com/mbowdish88/medweb-template) and serves as a reusable template.

**Live site:** [thevalvewire.com](https://thevalvewire.com)

## Running the Project

```bash
cd ~/projects/tavr-digest
source venv/bin/activate
python main.py              # Daily digest
python main.py --weekly     # Weekly summary
python main.py --podcast    # Podcast generation
```

Install dependencies: `pip install -r requirements.txt`

The pipeline runs daily at 6 AM Central via GitHub Actions. Manual trigger: `gh workflow run daily-digest.yml`

### Running the Website Locally

```bash
cd ~/projects/tavr-digest/site
npm install
npm run dev        # http://localhost:3000
npm run build      # production build
```

## Architecture

The pipeline follows: **Index Papers → Sources → Dedup → Summarize → Deliver → Write to site/**

### Sources (`sources/`)
Each module fetches from one external API/feed and returns a list of article dicts (`url`, `title`, `content`, `date`, `source`). Each has isolated error handling — one failing source doesn't block others.

| Module | Source | Notes |
|--------|--------|-------|
| `pubmed.py` | NCBI eUtils API (ESearch/EFetch) | 50 articles + 20 clinical trials |
| `preprints.py` | bioRxiv/medRxiv API | Filtered by relevance |
| `journals.py` | 12 RSS feeds (JACC, NEJM, Circulation, JAMA, Lancet, JTCVS, ATS, EJCTS, EHJ) | feedparser |
| `news.py` | Google News RSS | Site-specific searches (TCTMD, CV Business, CMS, SHN) |
| `regulatory.py` | FDA RSS feeds | Filtered by valve-related keywords |
| `trials.py` | ClinicalTrials.gov API v2 | 16 landmark trials monitored by NCT ID |
| `stocks.py` | yfinance | EW, MDT, ABT, BSX, AVR.AX — charts saved as PNGs |
| `social.py` | Nitter RSS bridges | 12 curated accounts — designed to fail gracefully |
| `financial.py` | SEC EDGAR 8-K filings + Google News financial RSS + yfinance news | |

### Processing (`processing/`)
| Module | Purpose |
|--------|---------|
| `dedup.py` | SQLite-backed deduplication (SHA256 URL hashing). Articles marked seen ONLY after successful email delivery. 90-day cleanup. Trials and stocks are live data, never deduped. |
| `summarizer.py` | Claude API (Sonnet 4, 16k token budget, 300s timeout) generates HTML newsletter. Includes major meeting detection for ACC, AHA, TCT, ESC, AATS, STS, EACTS. Also extracts structured article sidecar (key numbers, study design, journal) from raw inputs for downstream accuracy. |
| `weekly.py` | Compiles 7 days of daily digests into a single weekly summary. Saves both HTML and structured JSON sidecars. |

### Delivery (`delivery/`)
| Module | Purpose | Status |
|--------|---------|--------|
| `emailer.py` | SMTP email with HTML+plain-text via Jinja2 templates. Retries once with 10s delay. | Active |
| `website.py` | Writes structured JSON directly to `site/public/data/` (local filesystem, no API). Executive summary and key points preserve hyperlinks for verifiability. | Active |
| `site.py` | Publishes to GitHub Pages archive (`docs/`) | Active |
| `beehiiv.py` | Beehiiv API v2 newsletter delivery | Dormant |
| `substack.py` | Substack integration | Dormant |

### Website (`site/`)
The Valve Wire website — Next.js 16, App Router, Tailwind CSS 4, deployed on Vercel.

| Route | Purpose |
|-------|---------|
| `/` | Homepage — stock ticker, executive summary, key points, articles by section, podcast |
| `/archive` | Searchable/filterable article archive |
| `/weekly` | Latest weekly summary |
| `/podcast` | All podcast episodes with audio players + RSS |
| `/about` | Mission, editorial stance, AI disclosure |

Data flow: Pipeline writes JSON → `site/public/data/latest.json` + `site/public/data/digests/` → `git push` → Vercel auto-deploys.

All content pages use `dynamic = "force-dynamic"` for fresh data on each request.

### Podcast (`podcast/`)
Weekly podcast generation pipeline, triggered after the weekly digest completes.

| Module | Purpose |
|--------|---------|
| `scriptwriter.py` | Claude generates a two-host podcast script with journal hierarchy, meeting highlights, accuracy safeguards, and hallucination validation. Retries on JSON parse failure. 300s timeout. |
| `synthesizer.py` | OpenAI TTS voice synthesis (Nolan: "fable", Claire: "nova"). 3 retries with exponential backoff per segment. 120s timeout. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mbowdish88) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
