---
trigger: always_on
description: This is a product management workspace for {Your Company}'s product organisation. It contains strategy documents, research, project files, team notes, and process templates.
---

# {Your Company} Product Management Workspace

This is a product management workspace for {Your Company}'s product organisation. It contains strategy documents, research, project files, team notes, and process templates.

## Key context files

Before doing substantive work, read the relevant context files:

- `context/current-priorities.md` — current quarter focus areas, targets, and success metrics
- `data/funnel-context.md` — funnel definitions, core metrics, and how to interpret leading vs lagging indicators
- `pm-playbook/README.md` — entry point for PM process, templates, evidence standards, and training
- `context/product.md` — how the product works end-to-end
- `context/personas/customer-personas.md` and `partner-personas.md` — needs-based personas with prioritisation

<!-- SETUP: Add any additional context files here as you create them, e.g.:
- `strategy/roadmaps/YYYY-qN-roadmap.md` — current quarter roadmap
- `strategy/activation/activation-strategy.md` — activation strategy
- `context/brand/messaging/` — brand voice, tone, and terminology
-->

## About {Your Company}

<!-- SETUP: Replace this section with a 3-5 sentence summary of your company.
Include:
- What your company does and for whom
- Your business model (how you make money)
- Your key funnel stages (e.g. Sign-up → Activation → Purchase → Retention)
- Current strategic focus
- Key executives involved in product decisions

Example:
"{Your Company} is a B2B SaaS platform for [market]. We serve [N] customers
across [segments]. Revenue comes from [model]. The executive team includes
[CEO name] (CEO), [CPO name] (CPO), and [CTO name] (CTO).

Key funnel: **Leads → Trial → Activated → Paid → Retained**

Current strategic focus is activation — reducing the 70% drop-off during onboarding."
-->

## Conventions

- **{Language preference}** throughout all documents (e.g. UK English, US English)
- **Strategy documents** apply Reforge frameworks (activation moments, PNIP pyramid, psych framework, growth loops)
- **Project/experiment documents** follow the template in `pm-playbook/` — always include hypothesis, evidence, success metrics, guardrails, and risks
- **Goals should be outcome-oriented**, not deliverables. "Increase trial-to-paid from 4% to 10%" not "Launch new onboarding flow"
- **Tone** is direct, professional, and concise. No emojis unless explicitly asked. Avoid corporate fluff

## Folder structure

| Folder | Purpose |
|--------|---------|
| `context/` | Background info, current priorities, company context |
| `strategy/` | Roadmaps, OKRs, activation strategies, quarterly tracking |
| `projects/` | Active experiments and project documents |
| `insights/` | Market analysis, competitor research, qualitative research |
| `data/` | Metric definitions, saved queries, reports |
| `data/notebooks/` | Analyst Jupyter notebooks — templates, shared utilities, and automation outputs |
| `pm-playbook/` | PM process docs, templates, best practices |
| `team/` | Team member notes, feedback, agent state (gitignored — private) |
| `scripts/` | Utility scripts and automation infrastructure |

### Analyst notebooks

`data/notebooks/` contains Jupyter notebook templates for programmatic data analysis, run via papermill. The shared utilities module (`utils/pm_helpers.py`) provides data loading, z-score anomaly detection, chart styling, and standardised export helpers.

- **Templates** (`templates/`): `experiment-analysis`, `verify-claim`, `investigation-template`
- **Outputs** (`outputs/`, gitignored): Executed notebooks from papermill runs
- **Charts** (`data/reports/charts/`, gitignored): Generated PNGs, CSVs, and JSON files

Setup: `bash scripts/setup-notebooks.sh` (creates venv, installs deps, registers Jupyter kernel).

## Custom commands

Slash commands live in `.claude/commands/`. We use a naming convention to keep shared and personal commands distinct:

- **Shared commands** (no prefix) — useful for all PMs, documented here
- **Personal commands** (`{initials}-` prefix, e.g. `jk-`) — individual workflows, not expected to work for others

Each command file includes frontmatter with `owner`, `audience`, and `purpose` metadata.

### Shared commands

| Command | Arguments | Purpose |
|---------|-----------|---------|
| `/create-new-project` | — | Guided project/experiment doc creation with Socratic questioning |
| `/setup-agent` | `{initials}, {name}, {agent}` | Set up a new persistent agent — creates workspace, personalises definition, bootstraps context from live sources |
| `/interview-feedback` | `{name}, {level}, {type}` | Generate PM interview scorecard from meeting transcript |
| `/meeting-prep` | `{person name}` (optional) | Generate structured prep brief for a single upcoming meeting |
| `/meeting-schedule` | `{name1}, {name2}, ...` | Find mutual availability and schedule a meeting |
| `/investigate` | `{signal or question}` | Structured investigation: quant data + qual evidence + counter-evidence + hypothesis |
| `/daily-prep` | — | Full daily brief (today's focus + all meeting prep) |
| `/customer-reviews` | — | Weekly customer review summary from review platforms |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [motorway-sandbox/product-os](https://github.com/motorway-sandbox/product-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
