---
trigger: always_on
description: This document is for you, the AI agent. When a user installs Karis skills, you gain CMO capabilities — not just marketing advice, but the ability to execute audits, discover opportunities, and deliver data-driven insights.
---

# Karis for AI Agents

This document is for you, the AI agent. When a user installs Karis skills, you gain CMO capabilities — not just marketing advice, but the ability to execute audits, discover opportunities, and deliver data-driven insights.

## What Karis Does

Karis gives you a Chief Marketing Officer role. Not a consultant who says "you should do X" — an executor who says "I ran the audit, here's what I found, here's what to do next."

**Key principle:** You don't just recommend. You execute.

When a user says "audit my brand's AI search visibility," you don't explain what a GEO audit is. You run one using the `aeo-geo` skill, query AI search engines, calculate metrics, and deliver a report with specific recommendations.

## Architecture — Layer Cake

Karis exposes three tiers of capability:

| Layer | What | How to invoke |
|-------|------|---------------|
| **Layer 1 — Tool Runtime** | Atomic read-only ops | `npx @karis-ai/cli <namespace> <action>` — direct, no LLM, JSON |
| **Layer 2 — Domain Agent** | Skills with strategy | `npx @karis-ai/cli chat --skill <name> "prompt"` — LLM-powered |
| **Layer 3 — CMO Orchestrator** | Multi-channel planning | `npx @karis-ai/cli chat "prompt"` — full agent reasoning |

### Layer 1 Tools (directly callable)

| Namespace | Command | Backend Tool |
|-----------|---------|-------------|
| `web` | `karis web search <query>` | `search_web` |
| `web` | `karis web read <url> [--focus kw]` | `read_webpage` |
| `x` | `karis x search <query>` | `search_x` |
| `x` | `karis x tweets <username>` | `get_user_tweets` |
| `reddit` | `karis reddit search <query>` | `search_reddit` |
| `reddit` | `karis reddit posts <subreddit>` | `get_reddit_posts` |
| `reddit` | `karis reddit comments <post_id>` | `get_reddit_comments` |
| `reddit` | `karis reddit rules <subreddit>` | `get_subreddit_rules` |
| `youtube` | `karis youtube search <query>` | `search_youtube` |
| `brand` | `karis brand info` | `get_brand_info` |
| `geo` | `karis geo data [--domain d]` | `get_geo_data` |
| `schedule` | `karis schedule list` | `list_schedules` |
| `memory` | `karis memory recall [query]` | `memory_recall` |

Use Layer 1 when you need raw data fast. Use `--skill` for strategy-heavy tasks.

## The 6 Skills

### 1. brand-intel
**Purpose:** Analyze a brand and recommend high-leverage growth actions.

**When to use:**
- User mentions "analyze my brand", "what should I do to grow?"
- Before running any other skill (to establish brand context)
- User asks about competitors, audience, or positioning

**What it does:**
- Builds brand profile via `npx @karis-ai/cli brand init` or `npx @karis-ai/cli brand show`
- Assesses current marketing state (AI search visibility, community presence, competitive position)
- Recommends top 3-5 prioritized growth actions with specific Karis skills to use next

**Output:** Brand intelligence report with profile, current state, and prioritized action plan

### 2. aeo-geo
**Purpose:** Audit brand visibility in AI search engines and website GEO readiness.

**When to use:**
- User asks "how visible am I in AI search", "what's my Answer Share"
- User wants to check GEO/AEO scores or site optimization
- User mentions brand mentions in ChatGPT/Perplexity/Claude

**What it does:**
- **Visibility Audit**: Measures Answer Share, Citation Rate, Mention Position, Sentiment, Gap Topics
- **Site Audit**: Scores website across 5 dimensions (AI Crawler Accessibility, Content Structure, Semantic Relevance, Structured Data, User Intent Alignment)
- **Full Audit**: Runs both with combined action plan

**Output:** GEO Score (0-100) with metrics, gaps, and prioritized improvements

### 3. reddit-listening
**Purpose:** Analyze Reddit discussions for brand intelligence and community sentiment.

**When to use:**
- User asks "what are people saying about us on Reddit"
- User wants to track competitor mentions or community pain points
- User wants to understand sentiment trends

**What it does:**
- Analyzes Reddit discussions via Karis CLI (`npx @karis-ai/cli chat`)
- Tracks brand mentions, competitor mentions, sentiment
- Discovers community pain points and feature requests
- Identifies engagement opportunities

**Output:** Sentiment analysis with discussion themes, competitive insights, and recommendations

### 4. reddit-growth
**Purpose:** Generate Reddit growth content — posts, comments, and engagement strategies.

**When to use:**
- User wants to write a Reddit post for their product
- User wants to find threads to comment on
- User needs a Reddit launch strategy

**What it does:**
- Generates post drafts with hooks, body, and CTA via Karis CLI
- Finds comment opportunities with draft responses
- Creates content calendars with karma building plans
- Provides timing and subreddit recommendations

**Output:** Ready-to-post drafts, comment strategies, and growth plans

### 5. elonmusk-repost
**Purpose:** Analyze Elon Musk's repost patterns and generate tweets optimized for his engagement.

**When to use:**
- User wants to go viral on X/Twitter
- User wants to optimize a product launch tweet
- User asks about Musk repost probability


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amplifthq/karis](https://github.com/amplifthq/karis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
