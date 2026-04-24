---
trigger: always_on
description: An AI-powered marketing automation platform for **My Baking Creations**, a family-owned Bay Area bakery (founded 2012, Daly City CA).
---

# Bakers Agent — Project Instructions

## What This Is
An AI-powered marketing automation platform for **My Baking Creations**, a family-owned Bay Area bakery (founded 2012, Daly City CA).
Two repos, one business:
- **This repo (`bakers-agent`):** ~41 Cloud Functions (GCP), Pub/Sub pipeline, React dashboard at bakers-agent.web.app
- **Website repo (`gugosf114/gugosf114.github.io`):** Static site at mybakingcreations.com (GitHub Pages), ~800 commits

## The Website (mybakingcreations.com)
- **Repo:** github.com/gugosf114/gugosf114.github.io
- **Stack:** Static HTML/CSS/JS, GitHub Pages, custom domain
- **Pages:** Home, About, Gallery (cakes/cookies/cake pops/cupcakes + sub-galleries: sculpted, realistic, wedding, hand-piped, printed), Corporate, Blog, Party Rentals, Contact, Order Form, Buy Now, city landing pages
- **SEO:** JSON-LD structured data (Bakery, FAQPage, LocalBusiness, AggregateRating), Open Graph, Twitter Cards, CSP headers
- **Analytics:** GA4 (G-KB96GDJ011)
- **Features:** Hero carousel, Instagram video feed, corporate logo grid, live activity feed widget, AI chatbot, consultation/party rentals widget, site search with typewriter effect, nav particles canvas
- **Fonts:** Fredoka One (headings) + Nunito (body)
- **Theme color:** #EC268F (pink)
- **Corporate clients:** Google, Meta, Microsoft, Salesforce, DocuSign, OpenAI, Instagram, PayPal, Gap, Alaska Airlines, Warriors, Twitch, Cooley, Stripe, Levi's, Kaiser
- **corporate.html:** Title/H1 leads with cakes, not cookies (updated 2026-03-05)
- **robots.txt:** Wix migration slugs (/services-5, /contact-3, /about-1, /gallery-7) are Disallowed to stop crawl budget starvation
- **Social:** Instagram, Facebook, Pinterest, Yelp
- **Contact:** (415) 568-8060, info@mybakingcreations.com, 1096 Wildwood Ave, Daly City, CA 94015
- **Footer:** Credits Anthropic Claude for AI features

## Architecture
- **Cloud Functions (GCP):** ~41 functions, Gen2, Python 3.11+ (deploy uses 3.12)
- **Pub/Sub pipeline:** State machine with fan-out, DLQ (`bakers-agent-dlq`), memory/learning loop
- **Shared library:** `shared/bakers_shared/` — 27 modules, synced to each function at deploy via `tools/sync_shared.py`
- **Dashboard UI:** React 18 + Tailwind + shadcn/ui + Recharts + Firebase, hosted on Firebase Hosting
- **Database:** Firestore (database: `bakers-agent-db`)
- **Smart Router:** `content-router-v1` uses Gemini REST API (NOT the SDK) to decide platforms and adapt captions
- **Service accounts:** `bakers-brain-sa` (decision-making), `bakers-poster-sa` (posting)
- **Autonomy system:** 3 levels — AUTONOMOUS, SUPERVISED, RECOMMEND — per track via `get_track_autonomy()`

## Key Shared Modules (`shared/bakers_shared/`)
- **Core clients:** `firestore.py` (singleton via `client()`, uses `FIRESTORE_DB`), `pubsubutil.py` (`get_publisher()`), `secrets.py` (`access_secret_text()` with `lru_cache`), `pubsub.py` (`Publisher` + `decode_pubsub()`)
- **LLM:** `llm.py` — REST API only, exports `GeminiConfig`, `generate_text()`, `generate_json()`, `generate_markdown_from_brief()`
- **Content:** `sentinel.py` (validation: word count, banned words, price detection), `poster_base.py` (multichannel posting pattern), `firestore_store.py` (delegates to `firestore.client()`)
- **External APIs:** `github.py`, `gbp.py`, `klaviyo.py`, `yelp_profiles.py`, `mcp_client.py`
- **Utilities:** `logging.py` (JSON for Cloud Logging), `rate_limit.py`, `hashing.py`, `ids.py`, `textutil.py`, `timeutil.py`, `events.py`, `http.py`, `config.py`, `env.py`, `autonomy.py`

## Pub/Sub Topics
- **SEO:** `seo-ingest-topic`, `seo-opportunities-topic`, `seo-brief-topic`, `seo-pr-topic`, `seo-feedback-topic`
- **Content delivery:** `content-router-topic` → fan-out to `content-post-{gbp,facebook,instagram,pinterest,apple,yelp}-topic`
- **Agent Dun:** `dun-scheduler-topic`, `dun-digest-topic`, `dun-curiosity-topic`, `dun-pope-topic`, `dun-seo-bridge-topic`
- **Analytics/Ads:** `analytics-snapshot-trigger`, `analytics-alerts-topic`, `ads-daily-scheduler-trigger`, `ads-jobs-topic`
- **Reviews:** `gbp-reviews-topic`, `gbp-review-responder-topic`
- **Other:** `ai-visibility-topic`, `brand-consistency-topic`, `commerce-feeds-topic`, `directory-feeds-topic`, `llms-txt-topic`, `attribution-topic`, `segment-rebuild-topic`, `gsc-jobs-topic`, `local-seo-feeder-topic`
- **DLQ:** `bakers-agent-dlq`

## Firestore Collections
- `dun_config/`, `dun_jobs/` — Agent Dun config and unified job model
- `seo_jobs/`, `seo_sources/` — SEO pipeline
- `ai_visibility/`, `pope_visibility_log/` — AI search monitoring + Pope agent
- `curiosity_log/`, `memory/` — Learning loop (confidence-weighted insights)
- `analytics_snapshots/` — GA4 daily snapshots
- `brand_consistency/` — NAP audit results
- `google_ads/` — Ads campaign data
- `gbp_reviews/` — Google Business Profile reviews
- `captions/` — Social media captions (read-only for UI)
- `airlock/` — Human-in-the-loop approval queue
- `dun_ui_triggers/` — UI-initiated agent triggers

## Environment Variables
- **Core:** `GCP_PROJECT`, `FIRESTORE_DB`, `SITE_URL`, `SITE_NAME`
- **Feature toggles:** `DISABLE_INGEST`, `DISABLE_POSTING`, `DISABLE_PR_CREATION`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gugosf114) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
