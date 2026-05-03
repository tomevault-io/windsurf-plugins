---
trigger: always_on
description: AdLoop MCP orchestration — Google Ads + GA4 + codebase intelligence
---


# AdLoop — AI Orchestration Rules

You have access to AdLoop MCP tools that connect Google Ads and Google Analytics (GA4) data. These rules teach you how to use them intelligently.

## Tool Inventory

### Diagnostics

| Tool | When to Use | Key Parameters |
|------|-------------|----------------|
| `health_check` | First thing to run when tools are failing — tests OAuth token, GA4, and Ads connectivity | (none) |

**If health_check reports auth errors:** Tell the user to delete `~/.adloop/token.json` and re-run any tool to trigger re-authorization. If tokens keep expiring weekly, the GCP consent screen needs to be published from "Testing" to "In production".

### GA4 Read Tools

| Tool | When to Use | Key Parameters |
|------|-------------|----------------|
| `get_account_summaries` | First-time discovery — find which GA4 properties exist | (none — uses config) |
| `run_ga4_report` | Any analytics question — sessions, users, conversions, page performance | `dimensions`, `metrics`, `date_range_start`, `date_range_end`, `limit` |
| `run_realtime_report` | After code deploys — verify tracking fires correctly | `dimensions`, `metrics` |
| `get_tracking_events` | Understanding what events are configured and their volume | `date_range_start`, `date_range_end` |

### Google Ads Read Tools

| Tool | When to Use | Key Parameters |
|------|-------------|----------------|
| `list_accounts` | First-time discovery — find which Ads accounts exist | (none — uses MCC from config) |
| `get_campaign_performance` | Campaign-level metrics — impressions, clicks, cost, conversions | `date_range_start`, `date_range_end` |
| `get_ad_performance` | Ad copy analysis — which headlines/descriptions work | `date_range_start`, `date_range_end` |
| `get_keyword_performance` | Keyword analysis — quality scores, competitive metrics | `date_range_start`, `date_range_end` |
| `get_search_terms` | Find negative keyword opportunities and understand user intent | `date_range_start`, `date_range_end` |
| `get_negative_keywords` | List direct campaign-level negative keywords (not inside SharedSets) | `campaign_id` (optional) |
| `get_negative_keyword_lists` | List all shared negative keyword lists — names, IDs, status, keyword count | (none) |
| `get_negative_keyword_list_keywords` | List the keywords inside a specific shared negative keyword list | `shared_set_id` (required) |
| `get_negative_keyword_list_campaigns` | List which campaigns a shared negative keyword list is attached to | `shared_set_id` (optional) |
| `get_recommendations` | Google's auto-generated recommendations with estimated impact — budget, keyword, bid strategy, ad copy suggestions | `recommendation_types` (optional filter), `campaign_id` (optional) |
| `get_pmax_performance` | Performance Max campaign metrics with network breakdown + asset group ad strength | `date_range_start`, `date_range_end` |
| `get_asset_performance` | Per-asset details for PMax — field type, serving status, content. Use with `get_detailed_asset_performance` for quality signals | `campaign_id` (optional) |
| `get_detailed_asset_performance` | Top-performing asset combinations — which headline+description+image combos Google selects most | `campaign_id` (optional) |
| `get_audience_performance` | Audience segment metrics — remarketing, in-market, affinity, demographics | `date_range_start`, `date_range_end`, `campaign_id` (optional) |
| `run_gaql` | Custom queries not covered by other tools | `query`, `format` (table/json/csv) |

**Return format notes:**
- Ads read tools automatically compute `metrics.cost` and `metrics.cpa` from `metrics.cost_micros` — no manual division needed. `metrics.currency` contains the account's currency code (auto-detected).
- `metrics.average_cpc_amount` is also pre-computed where available.
- `get_ad_performance` returns full `headlines` and `descriptions` lists for RSAs.
- `get_recommendations` returns `estimated_improvement` per recommendation (potential minus base metrics) and `insights[]` that flag self-serving budget recommendations.
- PMax tools: `get_pmax_performance` returns `insights[]` flagging weak ad strength and zero-conversion asset groups. `segments.ad_network_type` includes MIXED — a Google catch-all for most PMax traffic. Full channel splits (Search vs YouTube vs Display vs Discover) are not available via the API.
- `get_asset_performance` returns `by_status` and `by_field_type` summaries. Note: per-asset performance labels (BEST/GOOD/LOW) are not available for PMax assets in API v23. Use `get_detailed_asset_performance` for quality signals via top combinations.
- `get_audience_performance` works for campaigns with explicit audience targeting. PMax audience targeting is automatic and may not appear in this report.

### Cross-Reference Tools (GA4 + Ads combined)

| Tool | When to Use | Key Parameters |
|------|-------------|----------------|
| `analyze_campaign_conversions` | "What's my real CPA?", paid vs organic comparison, GDPR gap analysis | `date_range_start`, `date_range_end`, `campaign_name` (optional filter) |
| `landing_page_analysis` | "Which landing pages convert?", identify pages with traffic but no conversions | `date_range_start`, `date_range_end` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kLOsk/adloop](https://github.com/kLOsk/adloop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
