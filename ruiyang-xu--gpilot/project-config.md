---
trigger: always_on
description: > **New here?** Read [README.md](README.md) first, then [docs/getting-started.md](docs/getting-started.md).
---

# GPilot — Financial Intelligence System

> **New here?** Read [README.md](README.md) first, then [docs/getting-started.md](docs/getting-started.md).
> **To customize**: Copy `.env.example` to `.env`, fill in your details, then run `bash scripts/customize.sh`.

## Identity

- **Operator**: YN
- **Organization**: Acme Ventures
- **Focus Sectors**: Software/SaaS, AI/ML, Fintech, Healthtech, Deeptech
- **Primary output**: Bilingual research reports (EN→CN), investment memos, portfolio analysis

## Directory Map

```
.
├── CLAUDE.md           ← You are here (system brain)
├── raw/                ← Source ingestion inbox (human drops files, LLM never modifies)
│   ├── deals/          ← Term sheets, deal flow, broker quotes
│   ├── research/       ← Articles, papers, conference notes
│   ├── market-intel/   ← News clips, analyst notes
│   ├── meetings/       ← Transcripts, notes
│   └── images/         ← Charts, diagrams for research
├── wiki/               ← LLM-compiled knowledge graph
│   ├── _index.md       ← Master TOC
│   ├── _summaries.md   ← 2-3 line summary of every document
│   ├── companies/      ← Per-company articles
│   ├── sectors/        ← Sector analysis articles
│   ├── deals/          ← Deal pipeline + per-deal status
│   ├── people/         ← Counterparties, network profiles
│   └── concepts/       ← Frameworks, structures, regulations
├── data/
│   ├── fund/           ← Workbooks (deal-tracker, portfolio, etc.)
│   ├── comps/          ← Sector comparable models
│   └── state/          ← JSON state (canonical SSOT) + schema
├── deals/              ← Per-company folders (notes, models, memos)
│   ├── {company-a}/    ├── {company-b}/    └── {company-c}/
├── learnings/          ← Agent processing experience (self-evolving)
│   ├── _index.md       ← Learnings overview
│   ├── preferences.md  ← User preferences (auto-learned)
│   ├── system.md       ← Cross-cutting system learnings
│   └── {agent}.md      ← Per-agent learnings (9 files)
├── agents/             ← 9 AI agent definitions
├── commands/           ← 22 slash command workflows
├── skills/             ← 7 skill domains (with reference docs)
├── templates/          ← 8 document templates
├── scheduled/          ← 10 core scheduled tasks
├── modules/            ← Optional modules (fund-ops, extras, integrations)
├── output/             ← Generated deliverables
├── research/           ← Research pipeline (wip/ + published/)
├── config/             ← Global config, rules, MCP settings
├── dashboard/          ← Next.js portfolio dashboard
├── scripts/            ← Setup and customization scripts
└── docs/               ← Getting started guide
```

## Data Sources

| Source | Access | Content |
|--------|--------|---------|
| **Local `data/`** | Direct read | Workbooks, sector comp models, state JSON |
| **Local `deals/`** | Direct read | Per-deal analysis, models, memos |
| **Google Drive** | MCP or symlink via `projects/` | Cloud-stored documents |
| **OneDrive** | MCP or symlink via `projects/` | Cloud-stored documents |

## Agents

| Agent | Role |
|-------|------|
| `deep-researcher` | Multi-source research via Perplexity MCP |
| `market-researcher` | TAM/SAM, competitive landscape, market maps |
| `financial-analyst` | Modeling, comps, valuations + public market data |
| `deal-sourcer` | Proactive opportunity discovery from GitHub, funding rounds, reports |
| `portfolio-monitor` | Portfolio news, KPI tracking, early warnings |
| `memo-writer` | Investment memos and financial documents |
| `editor` | Quality review, fact-checking, style enforcement |
| `translator` | EN↔CN bilingual (cultural adaptation) |
| `data-visualizer` | Charts, tables, frameworks for research |

> Additional agent in `modules/fund-ops/`: `legal-reviewer` (term sheet + side letter analysis)

## Commands

### Research Pipeline
| Command | Purpose |
|---------|---------|
| `/research` | Full research publication pipeline (8-step, with outline review) |
| `/research-fast-track` | Streamlined 6-step for pre-approved weekly/monthly publications |
| `/publish` | Format for WeChat + LinkedIn distribution |
| `/query` | Research a question against the wiki knowledge base |
| `/company-intel` | Deep company intelligence brief |

### Deal Sourcing & Analysis
| Command | Purpose |
|---------|---------|
| `/source-deals` | Proactive opportunity discovery from public sources |
| `/deal-screen` | Evaluate and score an investment opportunity |
| `/ic-memo` | Generate investment memo |
| `/portfolio-review` | Investment portfolio health check |
| `/board-prep` | Board/advisory meeting materials |
| `/market-data` | Public market data lookup |
| `/earnings-watch` | Earnings analysis |
| `/dcf` | Build institutional-quality DCF model with Bear/Base/Bull + sensitivity tables |
| `/portfolio-variance` | Per-company variance analysis — actuals vs budget, covenant compliance, KPI trends |
| `/founder-outreach` | Draft personalized founder cold email (Gmail drafts only, mandatory email dedup) |

### Knowledge Base & System
| Command | Purpose |
|---------|---------|
| `/ingest` | Process raw/ files into wiki (use `--full` for complete rebuild) |
| `/lint-wiki` | Wiki health check: stale data, broken links, contradictions |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ruiyang-xu/GPilot](https://github.com/ruiyang-xu/GPilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
