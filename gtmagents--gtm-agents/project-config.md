---
trigger: always_on
description: **GTM Agents** is an AI-powered automation platform for Go-To-Market teams. It provides 67 specialized plugins with 203 AI agents, 243 business skills, and 20+ workflow orchestrators designed for sales, marketing, growth, customer success, and revenue operations teams.
---

# CLAUDE.md - GTM Agents

## Project Overview

**GTM Agents** is an AI-powered automation platform for Go-To-Market teams. It provides 67 specialized plugins with 203 AI agents, 243 business skills, and 20+ workflow orchestrators designed for sales, marketing, growth, customer success, and revenue operations teams.

**Mission:** Save GTM professionals 15+ hours per week on repetitive busywork by automating prospecting, content creation, analytics, and campaign orchestration.

**Value Proposition:** Average value saved: $25,000+ per user per year.

---

## Repository Structure

```
gtm-agents/
├── .claude-plugin/
│   └── marketplace.json          # Plugin marketplace definition (67 plugins)
├── plugins/                       # 67 GTM-focused plugins
│   └── [plugin-name]/
│       ├── agents/               # Specialized AI agents (.md files)
│       ├── commands/             # Tools and workflows (.md files)
│       └── skills/               # Modular knowledge packages (SKILL.md)
├── workspace/                     # Agent working directory (see below)
├── docs/                         # Documentation
│   ├── agent-reference.md        # All 203 agents
│   ├── business-skills.md        # All 243 skills
│   ├── plugin-reference.md       # Plugin catalog
│   ├── usage-guide.md            # Commands and workflows
│   └── use-cases/                # Real-world examples
├── scripts/                      # Validation and scaffolding
│   ├── scaffold_asset.py         # Create new agents/commands/skills
│   ├── validate_marketplace.py   # Marketplace validation
│   └── smoke_test_plugins.py     # Plugin smoke tests
├── templates/                    # Asset templates
│   ├── agent.md
│   ├── command.md
│   └── skill.md
├── examples/                     # Example implementations
└── inventory.json                # Complete repository state
```

---

## Workspace Structure

The `workspace/` directory provides a consistent folder structure for all agent operations. **All agent outputs should be saved to the appropriate workspace folder.**

### Folder Structure

| Folder | Purpose |
|--------|---------|
| `clients/` | Client-specific folders and projects |
| `campaigns/` | Marketing and sales campaign materials |
| `content/` | Blog posts, articles, and content assets |
| `strategies/` | Content strategies and marketing plans |
| `research/` | Market research and data analysis |
| `social-media/` | Social media content and calendars |
| `email-campaigns/` | Email marketing materials and sequences |
| `reports/` | Performance reports and analytics |
| `templates/` | Reusable templates for deliverables |
| `assets/` | Brand assets, images, and media |
| `leads/` | Lead lists, prospect data, and enrichment |
| `pipelines/` | Sales pipeline snapshots and forecasts |
| `analytics/` | Analytics dashboards and data exports |
| `competitive-intel/` | Competitive research and battlecards |
| `personas/` | Buyer personas and ICP documentation |

### Agent Output Routing

When agents generate outputs, save to the appropriate folder:

| Agent Type | Output Folder |
|------------|---------------|
| Lead generation | `workspace/leads/` |
| Content creation | `workspace/content/` |
| Campaign planning | `workspace/campaigns/` |
| Research & analysis | `workspace/research/` |
| Performance data | `workspace/reports/` or `workspace/analytics/` |
| Competitive intel | `workspace/competitive-intel/` |
| Email sequences | `workspace/email-campaigns/` |
| Social content | `workspace/social-media/` |
| Strategy docs | `workspace/strategies/` |
| Persona development | `workspace/personas/` |

### Client Project Structure

Create subfolders for each client:
```
workspace/clients/
├── acme-corp/
│   ├── briefs/
│   ├── deliverables/
│   └── notes/
└── techstart-inc/
    ├── briefs/
    └── deliverables/
```

### Campaign Organization

Organize campaigns by quarter or initiative:
```
workspace/campaigns/
├── 2025-q1-product-launch/
├── 2025-q2-abm-enterprise/
└── ongoing-nurture/
```

### Naming Conventions

- **Lowercase with hyphens**: `q1-campaign-brief.md`
- **Include dates**: `2025-01-15-pipeline-report.csv`
- **Prefix drafts**: `draft-blog-post-ai-sales.md`
- **Prefix finals**: `final-case-study-acme.pdf`

### File Types

| Extension | Use For |
|-----------|---------|
| `.md` | Documents, briefs, strategies |
| `.csv` | Lead lists, data exports |
| `.json` | Structured data, configurations |
| `.xlsx` | Spreadsheets, reports |
| `.pdf` | Final deliverables |

---

## Core Concepts

### 1. Plugins (67 total)
Self-contained capability packages organized by GTM function:
- **Core Sales** (8): prospecting, pipeline, enablement, coaching, calls, enterprise, operations, account-management
- **Marketing** (13): content, email, social, SEO, paid-media, video, events, PR, brand, automation
- **Growth & Analytics** (9): experiments, revenue-analytics, customer-analytics, BI, PLG, pricing
- **Industry Verticals** (6): B2B SaaS, e-commerce, healthcare, financial services, EdTech, manufacturing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gtmagents/gtm-agents](https://github.com/gtmagents/gtm-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
