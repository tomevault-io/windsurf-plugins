---
trigger: always_on
description: Guidelines for AI agents contributing to the mureo codebase.
---

# AGENTS.md

Guidelines for AI agents contributing to the mureo codebase.

## Project Overview

**mureo** — your local-first AI ad ops crew. Find waste, audit changes, run ad accounts safely.

Works with Claude Code, Cursor, Codex & Gemini. mureo sits on top of the official ad-platform MCPs and gives your AI a strategy to follow, an outcome to be measured against, and an audit trail you can show to anyone — credentials never leave your machine.

mureo combines strategy context, workflow commands, and domain knowledge to help AI agents achieve marketing goals across platforms. Provides MCP tools for direct platform operations and workflow commands for strategy-driven ad operations via Claude Code slash commands. Designed for AI agents — no database, no LLM SDK, no web framework.

## Build & Test

```bash
pip install -e ".[dev]"
pytest tests/ -v
pytest tests/ --cov=mureo --cov-report=term-missing
```

## Architecture

```
mureo/
├── google_ads/          # Google Ads API client (Mixin composition)
│   ├── client.py        # GoogleAdsApiClient (main entry)
│   ├── mappers.py       # Response mapping to structured dicts
│   ├── _ads.py          # AdsMixin (RSA create/update/status/list)
│   ├── _ads_display.py  # DisplayAdsMixin (RDA create + RDAUploadError)
│   ├── _keywords.py     # KeywordsMixin (add/remove/suggest/diagnose)
│   ├── _analysis.py     # AnalysisMixin (auction/CPC/device/BtoB/RSA)
│   ├── _diagnostics.py  # DiagnosticsMixin (campaign diagnosis)
│   ├── _extensions.py   # ExtensionsMixin (sitelinks/callouts/conversions/targeting)
│   ├── _monitoring.py   # MonitoringMixin (anomaly detection/reporting)
│   ├── _creative.py     # CreativeMixin (LP analysis/message match)
│   ├── _media.py        # MediaMixin (image asset upload)
│   ├── _rsa_validator.py     # RSA ad text validator
│   ├── _rda_validator.py     # RDA input validator (display ads)
│   ├── _rsa_insights.py      # RSA asset performance insights
│   ├── _intent_classifier.py # Search term intent classification
│   └── _message_match.py     # Message match evaluator
├── meta_ads/            # Meta Ads API client (Mixin composition)
│   ├── client.py        # MetaAdsApiClient (main entry)
│   ├── mappers.py       # Response mapping
│   ├── _campaigns.py    # CampaignsMixin
│   ├── _ad_sets.py      # AdSetsMixin
│   ├── _ads.py          # AdsMixin
│   ├── _creatives.py    # CreativesMixin (image/carousel/collection/dynamic)
│   ├── _audiences.py    # AudiencesMixin
│   ├── _pixels.py       # PixelsMixin
│   ├── _insights.py     # InsightsMixin
│   ├── _analysis.py     # AnalysisMixin
│   ├── _catalog.py      # CatalogMixin (product catalogs/feeds)
│   ├── _conversions.py  # ConversionsMixin (Conversions API / CAPI)
│   ├── _hash_utils.py   # SHA-256 PII hashing for CAPI
│   ├── _leads.py        # LeadsMixin (lead forms/leads)
│   ├── _page_posts.py   # PagePostsMixin (page posts/boost)
│   ├── _instagram.py    # InstagramMixin (accounts/media/boost)
│   ├── _split_test.py   # SplitTestMixin (A/B split tests)
│   └── _ad_rules.py     # AdRulesMixin (automated rules)
├── search_console/      # Google Search Console API client (reuses Google OAuth2 credentials)
│   └── client.py        # SearchConsoleApiClient
├── mcp/                 # MCP server (Google Ads + Meta Ads + Search Console + Rollback + Analysis)
│   ├── server.py                          # MCP Server entry point (stdio-based)
│   ├── _helpers.py                        # Shared handler utilities
│   ├── tools_google_ads.py                # Google Ads tool definitions (aggregator)
│   ├── _tools_google_ads_*.py             # Tool definition sub-modules
│   ├── _handlers_google_ads.py            # Google Ads base handlers
│   ├── _handlers_google_ads_extensions.py # Extensions handlers
│   ├── _handlers_google_ads_analysis.py   # Analysis handlers
│   ├── tools_meta_ads.py                  # Meta Ads tool definitions (aggregator)
│   ├── _tools_meta_ads_*.py               # Tool definition sub-modules
│   ├── _handlers_meta_ads.py              # Meta Ads base handlers
│   ├── _handlers_meta_ads_extended.py     # Extended handlers
│   ├── _handlers_meta_ads_other.py        # Other handlers
│   ├── tools_search_console.py            # Search Console tool definitions
│   ├── _handlers_search_console.py        # Search Console handlers
│   ├── tools_rollback.py                  # rollback_plan_get / rollback_apply
│   ├── _handlers_rollback.py              # Rollback handlers (lazy-resolve dispatcher)
│   ├── tools_analysis.py                  # analysis_anomalies_check
│   └── _handlers_analysis.py              # Anomaly detector composition handler
├── cli/                 # Typer CLI (setup + auth + rollback inspection; ad operations are via MCP)
│   ├── main.py          # CLI entry point (`mureo` command)
│   ├── setup_cmd.py     # `mureo setup claude-code` / `cursor` / `codex` / `gemini`
│   ├── setup_codex.py   # Codex install-kit: MCP, credential guard, workflow command skills, shared skills
│   ├── setup_gemini.py  # Gemini extension manifest at ~/.gemini/extensions/mureo/
│   ├── auth_cmd.py      # `mureo auth setup` (+ `--web`) / `status` / `check-*`
│   ├── rollback_cmd.py  # `mureo rollback list` / `show` (inspection only; apply routes through MCP)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [logly/mureo](https://github.com/logly/mureo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
