---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Path Warning ⚠️

**CRITICAL**: The correct ecosystem path has changed!

**OLD/WRONG**: `C:\Users\Earth\BEDROT PRODUCTIONS\BEDROT DATA LAKE`  
**NEW/CORRECT**: `C:\Users\Earth\BEDROT PRODUCTIONS\bedrot-data-ecosystem`

Component paths:
- Data Lake: `bedrot-data-ecosystem\data_lake\`
- Data Warehouse: `bedrot-data-ecosystem\data_warehouse\`
- Dashboard: `bedrot-data-ecosystem\data_dashboard\`

Always verify you're in the correct directory before running commands!

## Project Overview

BEDROT Data Ecosystem is a production-grade analytics infrastructure for music industry data, consisting of three integrated components:

- **Data Lake** (`data_lake/`): Multi-zone ETL architecture for ingesting data from Spotify, TikTok, Meta Ads, DistroKid, Linktree, TooLost, YouTube, MailChimp
- **Data Warehouse** (`data_warehouse/`): SQLite-based analytical database with normalized tables for streaming, financial, and social media metrics  
- **Data Dashboard** (`data_dashboard/`): Real-time Next.js/FastAPI dashboard displaying 20+ KPIs with live WebSocket updates

## Zone Architecture

### Data Lake Zones
1. **1_landing/** - Raw data ingestion, timestamped files from extractors
2. **2_raw/** - Validated, immutable copies in NDJSON/CSV format
3. **3_staging/** - Cleaned, transformed, standardized data
4. **4_curated/** - Business-ready datasets for analytics/dashboards
5. **5_archive/** - Long-term storage (7+ year retention)
6. **6_automated_cronjob/** - Master pipeline orchestration scripts

### Service Implementation Status
| Service | Extractors | Cleaners | Priority | Critical Notes |
|---------|------------|----------|----------|----------------|
| TooLost | ✅ Multiple variants | ✅ All 3 | CRITICAL | JWT expires every 7 days! |
| Spotify | ✅ audience_extractor | ✅ All 3 | HIGH | Artists API |
| TikTok | ✅ 2 accounts | ✅ All 3 | HIGH | zonea0 + pig1987 |
| DistroKid | ✅ dk_auth | ✅ All 3 | MEDIUM | Streams + financials |
| Linktree | ✅ analytics_extractor | ✅ All 3 | MEDIUM | GraphQL |
| MetaAds | ✅ daily_campaigns | ✅ All 3 | LOW | Graph API |
| Instagram | ❌ Empty dirs | ❌ | - | Not implemented |
| YouTube | ❌ Empty dirs | ❌ | - | Not implemented |
| MailChimp | ❌ Empty dirs | ❌ | - | Not implemented |

## Important Context: Semi-Manual Authentication Design

### Authentication Philosophy
**CRITICAL**: The BEDROT Data Lake operates as a **semi-manual system by design**. This is not a limitation but a deliberate choice for compliance and security:

1. **Extractors NEVER authenticate automatically** - All authentication is handled manually by the user
2. **2FA Compliance** - All services use two-factor authentication, requiring human interaction
3. **Cookie Management is Convenience, Not Critical** - Expired cookies mean manual re-authentication, not system failure
4. **User-Driven Process** - The system respects service terms by requiring explicit user authentication

### Cookie Management Role
The cookie system exists to **reduce** manual work, not eliminate it:
- **Storage**: Preserves authentication state between runs
- **Monitoring**: Warns when cookies are expiring
- **Refresh Assistance**: Helps guide re-authentication when needed
- **NOT Critical Infrastructure**: Cookie failures are inconveniences, not emergencies

### Interpreting Cookie-Related Failures
When you see extraction failures due to cookies:
1. **This is EXPECTED behavior** - The system is working as designed
2. **Not a critical error** - Just means manual authentication is needed
3. **Part of the workflow** - Semi-manual means user intervention is normal
4. **Compliance-friendly** - Ensures we're not automating around security measures

## Development Commands

### Testing
```bash
# Run all tests with coverage
pytest -ra --cov=src --cov-report=term-missing

# Run tests for specific service
pytest tests/spotify/ -v

# Run cookie refresh tests
pytest tests/test_cookie_refresh.py -v
pytest tests/test_integration.py -v
pytest tests/test_e2e_scenarios.py -v
```

### Code Quality
```bash
# Format code
black src/
isort src/

# Lint code  
flake8 src/
mypy src/
```

### Data Pipeline Execution

#### New Semi-Manual Pipeline with Authentication (Recommended)
```bash
# Interactive mode - prompts for manual auth when needed
cd "C:\Users\Earth\BEDROT PRODUCTIONS\bedrot-data-ecosystem\data_lake"
cronjob\run_bedrot_pipeline.bat

# Automated mode - skips services needing auth
cronjob\run_bedrot_pipeline.bat --automated

# Alternative with data warehouse ETL
cronjob\run_pipeline_with_auth.bat
```

#### Legacy Pipeline Commands
```bash
# Run full data lake pipeline (Windows)
data_lake/cronjob/run_datalake_cron.bat

# Run pipeline without extractors (cleaners only)
data_lake/cronjob/run_datalake_cron_no_extractors.bat

# Manual Python execution requires PROJECT_ROOT environment variable
set PROJECT_ROOT=%cd%
python src/spotify/extractors/spotify_audience_extractor.py
```

#### Health Monitoring & Diagnostics
```bash
# Check pipeline health
python src\common\pipeline_health_monitor.py

# Check authentication status
python src\common\run_with_auth_check.py --check-only


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/blakedemarest) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
