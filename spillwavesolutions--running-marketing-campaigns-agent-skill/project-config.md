---
trigger: always_on
description: >
---


# Marketing Campaign Execution

Plan, execute, and measure digital marketing campaigns across content, social, email, and analytics.

## Contents

- [Quick Start](#quick-start)
- [Domain Reference Guide](#domain-reference-guide)
- [Scripts](#scripts)
- [Workflow Decision Tree](#workflow-decision-tree)
- [Multi-Domain Queries](#multi-domain-queries)
- [Campaign Validation Checklist](#campaign-validation-checklist)
- [Persona Adaptation](#persona-adaptation)
- [Boundaries](#boundaries)

## Quick Start

### Generate UTM Parameters
```
Source: where traffic originates (google, facebook, newsletter)
Medium: how it arrives (cpc, email, social, organic)
Campaign: initiative name (spring-sale-2025, product-launch)

Format: lowercase, hyphens, no spaces
Input: "Spring Sale 2025" → Output: "spring-sale-2025"
Input: "Q1 Launch Campaign" → Output: "q1-launch-campaign"

Example: ?utm_source=linkedin&utm_medium=social&utm_campaign=q1-launch
```

### Create Email Sequence
1. Welcome (immediate): Set expectations, deliver promised value
2. Value (day 2-3): Best content or quick win
3. Engagement (day 5-7): Encourage reply or action
4. Offer (day 10): Clear CTA with incentive

### Plan Content Calendar
Essential fields: Title, Target keyword, Funnel stage (TOFU/MOFU/BOFU), Format, Owner, Publish date, Distribution channels.

### Check Campaign Performance
Primary metrics by channel:
- Email: Open rate (43% avg), CTR (2% avg), Conversion rate
- Social: Engagement rate, Reach, Click-through
- Paid: ROAS, CPA, CTR
- Content: Traffic, Time on page, Conversions

## Domain Reference Guide

| Need | Reference | When to Load |
|------|-----------|--------------|
| Plan content strategy | [content-strategy.md](references/content-strategy.md) | Topic clusters, calendars, funnel mapping, repurposing |
| Execute social media | [social-media.md](references/social-media.md) | Platform tactics, posting times, engagement benchmarks |
| Build email campaigns | [email-marketing.md](references/email-marketing.md) | Sequences, subject lines, segmentation, deliverability |
| Track campaigns | [utm-tracking.md](references/utm-tracking.md) | UTM formatting, naming conventions, GA4 alignment |
| Measure performance | [analytics-measurement.md](references/analytics-measurement.md) | KPIs, GA4 setup, attribution, ROI calculations |
| Launch products | [gtm-tools.md](references/gtm-tools.md) | GTM frameworks, positioning, tool selection |
| Define brand voice | [brand-guidelines.md](references/brand-guidelines.md) | Voice dimensions, tone, messaging framework, terminology |
| Optimize for search | [seo-optimization.md](references/seo-optimization.md) | Technical SEO, on-page, content SEO, link building, E-E-A-T |
| Optimize for AI | [geo-optimization.md](references/geo-optimization.md) | GEO, LLMO, AEO, AI Overviews, chatbot visibility |

## Scripts

Python utilities for campaign automation:

| Script | Purpose | Usage |
|--------|---------|-------|
| [utm_tools.py](scripts/utm_tools.py) | UTM generation, validation, batch processing, QR codes | `python utm_tools.py generate --source facebook --medium paid-social --campaign q1-launch` |
| [brand_checker.py](scripts/brand_checker.py) | Brand voice compliance, readability scoring, banned words | `python brand_checker.py check --file copy.txt` |

### Script Quick Reference

**Generate and validate UTMs:**
```bash
# Generate UTM parameters
python scripts/utm_tools.py generate -s facebook -m paid-social -c spring-2025

# Build complete tracking URL
python scripts/utm_tools.py build -u https://example.com -s email -m newsletter -c q1-launch

# Validate existing URL
python scripts/utm_tools.py validate -u "https://example.com?utm_source=email&utm_medium=cpc"

# Batch process from CSV
python scripts/utm_tools.py batch -f campaigns.csv -u https://example.com -o tracking.csv

# Check GA4 channel mapping
python scripts/utm_tools.py ga4-check -s facebook -m paid-social
```

**Check brand compliance:**
```bash
# Full compliance check
python scripts/brand_checker.py check --file marketing_copy.txt

# Check readability score
python scripts/brand_checker.py readability --text "Your marketing copy here"

# Find banned words
python scripts/brand_checker.py banned --file email_draft.txt

# Full audit with JSON output
python scripts/brand_checker.py full-audit --file campaign.txt --output report.json
```

## Workflow Decision Tree

**What does the user need?**

```
Creating or planning content?
├─ Yes → content-strategy.md
│        • Topic clusters, pillar pages
│        • Content calendars (annual/quarterly/weekly)
│        • TOFU/MOFU/BOFU mapping
│        • Repurposing workflows
└─ No ↓

Platform-specific social guidance?
├─ Yes → social-media.md
│        • Instagram, LinkedIn, TikTok, X, Facebook
│        • Posting cadence and timing
│        • Algorithm priorities
│        • Engagement benchmarks
└─ No ↓

Email campaigns or sequences?
├─ Yes → email-marketing.md
│        • Welcome, drip, re-engagement sequences
│        • Subject line optimization
│        • Segmentation strategies
│        • Deliverability requirements
└─ No ↓

UTM parameters or tracking URLs?
├─ Yes → utm-tracking.md + scripts/utm_tools.py
│        • Parameter formatting rules
│        • Naming conventions
│        • GA4 channel alignment
│        • Dynamic parameters for ads

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SpillwaveSolutions/running-marketing-campaigns-agent-skill](https://github.com/SpillwaveSolutions/running-marketing-campaigns-agent-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
