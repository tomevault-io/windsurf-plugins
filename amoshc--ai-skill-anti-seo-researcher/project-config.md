---
trigger: always_on
description: >
---


# Anti-SEO Deep Consumer Researcher

> Detailed rules, scoring criteria, and category examples: see `references/SKILL_REFERENCE.md`

## Architecture Overview

**Language Detection → AI Category Adaptation → AI Multi-layer Search (forum posts + e-commerce reviews + social comment sections) → Script Scoring → AI Semantic Analysis → Dynamic Multi-dimensional Scoring → Report**

- **Language & Region Layer**: Detect user's language from query, generate region-specific platform config, search templates, and keyword dictionaries
- **Category Layer**: AI generates `category_profile` JSON (evaluation dimensions/weights/pain point keywords/safety risks/platform weights/e-commerce search strategy)
- **Search Layer** (3-tier data sources):
  - **L1 E-commerce Review Layer** (highest priority): Indirect search for real buyer reviews (e.g., Amazon reviews, JD follow-up reviews, depending on region)
  - **L2 Social Comment Section Layer** (second priority): Search for "debunking" feedback in comment sections of promotional posts
  - **L3 Forum Post Layer** (traditional): AI uses `web_search` + `site:` for targeted community searches
- **Scoring Layer**: `credibility_scorer.py` (regex pre-filter + category signal injection + data source tier weighting) → `ai_credibility_analyzer.py` (AI deep analysis for gray zone 30-85 scores)
- **Multi-dimensional Scoring**: `brand_scorer.py` (dimensions/weights from profile, safety capping is category-adaptive)
- **Report Layer**: `generate_report.py` (dynamic table headers + data source distribution stats, from profile dimension definitions)

## Multi-language & Multi-region Adaptation

**Core Principle**: This tool adapts to any language and region. The AI detects the user's language from their query and generates ALL region-specific configurations dynamically in the `category_profile`.

### Language Detection Rules

1. Detect the language of the user's query (Chinese, English, Japanese, Korean, etc.)
2. Infer the target market/region from context (e.g., Chinese query → China market; English query about "best vacuum" → likely US/UK market; Japanese query → Japan market)
3. ALL subsequent search queries, keywords, and report text MUST match the detected language and region
4. If the user explicitly mentions a region (e.g., "available in the UK", "sold on Amazon Japan"), use that region regardless of query language

### Regional Platform Mapping

The AI MUST generate appropriate platform configurations based on the detected region. Below are reference mappings (the AI should adapt these based on actual availability and relevance):

**China (zh-CN)**:
| Tier | Platforms | Examples |
|------|-----------|----------|
| L1 E-commerce | JD.com, Taobao, Pinduoduo | Review aggregation posts, follow-up reviews |
| L2 Social Comments | Xiaohongshu, Zhihu | "Debunking" comments under promotional posts |
| L3/L4 Forums | V2EX, Chiphell, NGA, Baidu Tieba, SMZDM, Douban, Bilibili | Community discussions, in-depth reviews |

**United States / English-speaking (en-US)**:
| Tier | Platforms | Examples |
|------|-----------|----------|
| L1 E-commerce | Amazon, Best Buy, Walmart | Verified purchase reviews, long-term reviews |
| L2 Social Comments | Reddit, YouTube comments | Comment sections debunking sponsored content |
| L3/L4 Forums | Reddit (subreddits), Head-Fi, AVSForum, Wirecutter comments, Slickdeals | Community discussions, enthusiast reviews |

**Japan (ja-JP)**:
| Tier | Platforms | Examples |
|------|-----------|----------|
| L1 E-commerce | Amazon.co.jp, Rakuten, Kakaku.com | Purchase reviews, price comparison reviews |
| L2 Social Comments | Twitter/X, note.com comments | Real user feedback under promotional content |
| L3/L4 Forums | Kakaku.com forums, 5ch, Price.com | Community discussions, expert reviews |

**South Korea (ko-KR)**:
| Tier | Platforms | Examples |
|------|-----------|----------|
| L1 E-commerce | Coupang, Naver Shopping | Purchase reviews |
| L2 Social Comments | Naver Blog comments, Instagram | Real feedback |
| L3/L4 Forums | DC Inside, Naver Cafe, Clien | Community discussions |

**Europe (various)**:
| Tier | Platforms | Examples |
|------|-----------|----------|
| L1 E-commerce | Amazon (regional), Trustpilot | Purchase reviews, trust scores |
| L2 Social Comments | Reddit, YouTube, regional social | Comment section feedback |
| L3/L4 Forums | Regional forums, Reddit (subreddits) | Community discussions |

### Regional Regulatory Authorities

Safety event searches must include the correct regulatory bodies for the target region:

| Region | Regulatory Bodies |
|--------|-------------------|
| China | SAMR (State Administration for Market Regulation), CFDA |
| US | FDA, CPSC, FTC |
| EU | EFSA, ECHA, national agencies |
| Japan | MHLW, CAA, NITE |
| South Korea | MFDS, KCA |

### Regional Marketing Signal Adaptation

Each region has different marketing manipulation patterns. The AI MUST generate region-appropriate marketing signals in `category_profile`:

**China**: SEO manipulation keywords (e.g., marketing buzzwords, "zhong cao/ba cao" patterns), fake review indicators, WeChat marketing patterns
**US/UK**: Affiliate link indicators, sponsored content disclaimers, Amazon vine/incentivized review patterns, influencer disclosure signals

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AmosHc/ai-skill-anti-seo-researcher](https://github.com/AmosHc/ai-skill-anti-seo-researcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
