---
trigger: always_on
description: Manage advertising campaigns across Google Ads, Meta Ads, LinkedIn Ads, and TikTok Ads using the Adspirer MCP server (175+ tools across Google Search/PMax/Display/Demand Gen/YouTube, Meta image/video/carousel/lead-gen, LinkedIn sponsored content/carousel/lead-gen with campaign groups, and TikTok in-feed/Spark/Carousel/App Promotion).
---

# Adspirer — Ad Campaign Management

Manage advertising campaigns across Google Ads, Meta Ads, LinkedIn Ads, and TikTok Ads using the Adspirer MCP server (175+ tools across Google Search/PMax/Display/Demand Gen/YouTube, Meta image/video/carousel/lead-gen, LinkedIn sponsored content/carousel/lead-gen with campaign groups, and TikTok in-feed/Spark/Carousel/App Promotion).

## Quick Start

1. Call `get_connections_status` to see which ad platforms are connected
2. If no platforms connected, direct the user to https://adspirer.ai/connections to link accounts
3. For campaign performance, call the platform-specific performance tool
4. Present results in tables with key metrics

## When to Use These Tools

Activate when the user:

- Asks about ad campaign performance ("How are my Google Ads doing?")
- Wants to research keywords ("Find keywords for my plumbing business")
- Needs to create campaigns ("Launch a Google Search campaign for...")
- Wants budget optimization ("Where am I wasting ad spend?")
- Mentions advertising platforms (Google Ads, Meta, LinkedIn, TikTok)
- Asks about ad accounts or connections ("Which ad platforms are connected?")
- Wants to manage creatives, audiences, or ad extensions
- Asks about monitoring or scheduled reports

## Required Workflow

**Follow these steps in order. Do not skip steps.**

### Step 1: Check Connected Platforms

Always start here before any ad operation:

- Call `get_connections_status`
- Shows connected platforms, primary/secondary accounts, account IDs
- If the target platform is not connected, direct the user to https://adspirer.ai/connections

### Step 2: Identify the Task

| User goal | Key tools |
|-----------|-----------|
| View campaign metrics | `get_campaign_performance`, `get_meta_campaign_performance`, `get_linkedin_campaign_performance`, `get_tiktok_campaign_performance` |
| Cross-platform overview | Call each platform's performance tool, present side-by-side |
| Find keywords | `research_keywords`, `analyze_search_terms` |
| Create a campaign | Platform-specific creation tools (always PAUSED) |
| Reduce wasted spend | `analyze_wasted_spend`, `analyze_meta_wasted_spend`, `analyze_linkedin_wasted_spend` |
| Optimize budgets | `optimize_budget_allocation`, `optimize_meta_budget`, `optimize_linkedin_budget` |
| Switch accounts | `switch_primary_account` |
| Check ad fatigue | `detect_meta_creative_fatigue`, `analyze_linkedin_creative_performance` |
| Understand audiences | `get_meta_audience_insights`, `get_linkedin_audience_insights`, `search_audiences` |
| Manage ad extensions | `add_sitelinks`, `add_callout_extensions`, `add_structured_snippets` |
| Set up monitoring | `create_monitor`, `list_monitors` |
| Schedule reports | `schedule_brief`, `generate_report_now` |
| Explain anomalies | `explain_performance_anomaly`, `explain_meta_anomaly`, `explain_linkedin_anomaly` |

### Step 3: Execute Tools

Follow these workflow patterns. Always read first (performance, status), then act (create, optimize).

### Step 4: Summarize and Recommend

Present results in tables with key metrics. Highlight top and underperforming items. Propose actionable next steps.

## Performance Analysis

- **Google Ads:** `get_campaign_performance` — params: `lookback_days` (7/30/60/90, default 30), optional `customer_id`
- **Meta Ads:** `get_meta_campaign_performance` — params: `lookback_days`, optional `ad_account_id`
- **LinkedIn Ads:** `get_linkedin_campaign_performance` — params: `lookback_days`
- **TikTok Ads:** `get_tiktok_campaign_performance` — params: `lookback_days`

Present: impressions, clicks, CTR, spend, conversions, cost/conversion, ROAS. Default to 30-day lookback.

## Cross-Platform Performance Dashboard

When the user asks for overall performance, a weekly review, or cross-platform comparison:

1. Call `get_connections_status` to identify active platforms
2. For each connected platform, pull performance data
3. For each platform, pull waste analysis
4. Present a unified scorecard:

| Platform | Campaigns | Spend | CTR | CPA | ROAS | Waste | Health |
|----------|-----------|-------|-----|-----|------|-------|--------|
| Google   | ...       | ...   | ... | ... | ...  | ...   | ...    |
| LinkedIn | ...       | ...   | ... | ... | ...  | ...   | ...    |
| Meta     | ...       | ...   | ... | ... | ...  | ...   | ...    |
| **Total**| ...       | ...   |     |     |      | ...   |        |

5. Highlight best and worst performing platforms and campaigns
6. Recommend top 3 actions across all platforms

## Keyword Research

1. `research_keywords` — get search volumes, CPC, competition data
2. `analyze_search_terms` — see what users actually search for
3. Use findings to inform campaign creation or optimization
4. Present keyword opportunities in a table with volume, CPC, and competition

## Campaign Creation

All campaigns are created in **PAUSED** status. Follow this order:

1. **Research:** keywords, competitors, audiences

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amekala/ads-mcp](https://github.com/amekala/ads-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
