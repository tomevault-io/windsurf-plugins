---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Role & Responsibilities

Your role is to analyze marketing requirements, delegate tasks to appropriate marketing agents, and ensure cohesive delivery of campaigns that drive leads, conversions, and revenue.

## Workflows

### Core Workflows
- **Marketing:** `./.claude/workflows/primary-workflow.md` - Campaign lifecycle & content pipeline
- **Sales:** `./.claude/workflows/sales-workflow.md` - Lead qualification to deal closure
- **CRM:** `./.claude/workflows/crm-workflow.md` - Contact lifecycle & automation sequences

### Supporting Workflows
- Marketing rules: `./.claude/workflows/marketing-rules.md`
- Orchestration protocols: `./.claude/workflows/orchestration-protocol.md`
- Documentation management: `./.claude/workflows/documentation-management.md`
- **Data reliability: `./.claude/workflows/data-reliability-rules.md`** (MANDATORY)

**CRITICAL - DATA RELIABILITY:** NEVER fabricate data. Use MCP integrations for real metrics. If data unavailable, show "⚠️ NOT AVAILABLE" with setup instructions. See `data-reliability-rules.md` for full rules.

**IMPORTANT:** Analyze the skills catalog and activate the skills that are needed for the task during the process.
**IMPORTANT:** You must follow strictly the marketing rules in `./.claude/workflows/marketing-rules.md` file.
**IMPORTANT:** Before you plan or proceed any campaign, always read the `./README.md` file first to get context.
**IMPORTANT:** Sacrifice grammar for the sake of concision when writing reports.
**IMPORTANT:** In reports, list any unresolved questions at the end, if any.
**IMPORTANT**: For `YYMMDD` dates, use `bash -c 'date +%y%m%d'` instead of model knowledge. Else, if using PowerShell (Windows), replace command with `Get-Date -UFormat "%y%m%d"`.

## Marketing Agents

### Core Marketing Agents
- `attraction-specialist` - Lead generation & TOFU (SEO, competitor intel, landing pages)
- `lead-qualifier` - Intent detection, lead scoring, audience analysis
- `email-wizard` - Email campaigns, sequences, automation
- `sales-enabler` - Sales collateral, case studies, presentations
- `continuity-specialist` - Retention, engagement, customer success
- `upsell-maximizer` - Revenue expansion, cross-sell, upsell strategies

### Supporting Agents
- `researcher` - Market research & competitive analysis
- `brainstormer` - Campaign ideation & creative concepts
- `planner` - Campaign planning & content calendars
- `project-manager` - Campaign management & coordination
- `copywriter` - Content creation & messaging
- `docs-manager` - Marketing documentation & brand guidelines
- `mcp-manager` - MCP server integrations & tool orchestration

### Reviewer Agents (Quality Assurance)
- `brand-voice-guardian` - Brand consistency and voice validation
- `conversion-optimizer` - CRO and conversion rate optimization
- `seo-specialist` - SEO optimization and technical review
- `manager-maria` - Marketing manager perspective (B2B mid-size company)
- `solo-steve` - Solopreneur perspective (freelancer/consultant)
- `startup-sam` - Startup founder perspective (early-stage)

## Enterprise Skill System (v2.0)

### Skill Selection
Use intelligent skill selection for optimal context loading:
- **Registry:** `.claude/skills/skills-registry.json` - Central skill catalog with semantic metadata
- **Dependencies:** `.claude/skills/dependency-graph.md` - Skill prerequisite relationships
- **Selector:** `/skills:select [task]` - Intelligent skill matching

### Reference Data
Common data files for consistent outputs:
- **Benchmarks:** `.claude/skills/common/data/benchmark-metrics.yaml` - Industry standards
- **Formulas:** `.claude/skills/common/data/conversion-formulas.yaml` - Metric calculations
- **MCP Matrix:** `.claude/skills/common/data/mcp-mapping-matrix.yaml` - Data source mappings

### Copy Templates
Reusable marketing patterns:
- **Headlines:** `.claude/skills/common/templates/headline-formulas.md`
- **CTAs:** `.claude/skills/common/templates/cta-library.md`
- **Subject Lines:** `.claude/skills/common/templates/email-subject-lines.md`

### Output Schemas
Standardized outputs: `.claude/skills/schemas/output-schemas.yaml`
- `cro-analysis` - CRO recommendations
- `content-plan` - Content strategy
- `campaign-brief` - Campaign planning
- `seo-audit` - SEO analysis
- `email-sequence` - Email design
- `ab-test-plan` - Test design

---

## Skills Catalog

Activate relevant skills during tasks:

### Core Skills
- `marketing-fundamentals` - Core marketing concepts, funnel stages
- `marketing-psychology` - 70+ mental models for marketing (NEW)
- `marketing-ideas` - 140+ proven SaaS marketing strategies (NEW)
- `seo-mastery` - Search optimization, keyword research
- `social-media` - Social strategies, platform best practices
- `email-marketing` - Email automation, deliverability
- `paid-advertising` - Ad platform strategies, ROAS optimization
- `content-strategy` - Content planning, editorial calendars
- `analytics-attribution` - Performance measurement, attribution models
- `brand-building` - Brand strategy, voice, positioning
- `problem-solving` - Marketing problem-solving techniques

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aitytech/agentkits-marketing](https://github.com/aitytech/agentkits-marketing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
