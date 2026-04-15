---
trigger: always_on
description: This file provides guidance to Claude Code (and future AI assistants) when working with this repository.
---

# CLAUDE.md - W.I.Z.A.R.D.

## Work Item Zero-touch Automation for Repo Delivery

This file provides guidance to Claude Code (and future AI assistants) when working with this repository.

---

## Project Overview

**WIZARD** is a DevOps automation agent that:
- Ingests work items from Azure DevOps and ServiceNow
- Matches requests to Skills (bundled SOPs)
- Generates IaC changes using Azure OpenAI Agents SDK
- Creates PRs across configured repositories
- Integrates with Flux GitOps workflow

## Project Status

**Current Phase**: Context Engineering - Feature Generation

Following the Context Engineering Methodology:
- [x] Stage 1: Ideation (complete)
- [x] Stage 2: Deep Research (complete - Gemini, Claude, ChatGPT)
- [x] Stage 3: Document Parsing (complete - SYNTHESIS.md)
- [x] Stage 4: Architecture Generation (complete - Agent Skills integrated)
- [ ] Stage 5: Feature Generation (next)
- [ ] Stage 6: Implementation

## Key Documents

| Document | Purpose |
|----------|---------|
| `docs/PROJECT_SUMMARY.md` | Goals, scope, success criteria |
| `docs/ARCHITECTURE.md` | Technical design and component breakdown |
| `docs/BUSINESS_CASE.md` | Value proposition and roadmap |
| `docs/RISKS.md` | Risk assessment and mitigations |
| `docs/IMPLEMENTATION_REALITY.md` | **Phase 0 toolkit requirements** |
| `docs/BUILD_VS_BUY.md` | **Build vs. buy analysis with market research** |
| `docs/research/SYNTHESIS.md` | Synthesized findings from all research |
| `CONTEXT_ENGINEERING_METHODOLOGY.md` | Development methodology |

### External Dependencies

| Resource | Purpose |
|----------|---------|
| [agent-toolkit](https://github.com/brendanbecker/agent-toolkit) | Phase 0 implementation repository |
| [featmgmt subagent-patterns](https://github.com/brendanbecker/featmgmt/tree/master/subagent-patterns) | Framework-agnostic agent specifications for Phase 0 |

## Repository Structure

```
wizard/
├── CLAUDE.md                              # This file
├── CONTEXT_ENGINEERING_METHODOLOGY.md     # Methodology guide
├── docs/
│   ├── PROJECT_SUMMARY.md                 # Project overview
│   ├── ARCHITECTURE.md                    # Technical architecture (v1.1)
│   └── research/
│       ├── DEEP_RESEARCH_PROMPT.md        # Research prompt for Stage 2
│       ├── SYNTHESIS.md                   # Synthesized research findings
│       ├── gemini_research.md             # Gemini deep research
│       ├── claude_research.md             # Claude deep research
│       └── chatgpt_research.pdf           # ChatGPT deep research
├── config/
│   ├── repos.yaml                         # (pending) Repo registry
│   ├── providers.yaml                     # (pending) Git provider configs
│   └── settings.yaml                      # (pending) General settings
├── skills/
│   └── _template/                         # Agent Skills standard template
│       ├── SKILL.md                       # Frontmatter + instructions
│       ├── scripts/validate.sh            # Validation script
│       ├── references/                    # Example documentation
│       └── assets/                        # Jinja2 templates
├── src/
│   ├── adapters/                          # (pending) Entry point adapters
│   ├── providers/                         # (pending) Git providers
│   ├── classifier/                        # (pending) Request classifier
│   ├── executor/                          # (pending) Skill executor
│   └── core/                              # (pending) Shared utilities
└── feature-management/                    # (pending) featmgmt integration
```

## Related Tools

WIZARD is part of a themed tool ecosystem:

| Class | Name | Role |
|-------|------|------|
| Ranger | ConfigTool | Config management |
| Sorcerer | ResourceTool | Resource management |
| Cleric | IncidentTool | Incident response (future) |
| **Wizard** | **WIZARD** | **DevOps automation** |

## Technology Stack

| Component | Technology |
|-----------|------------|
| Language | Python 3.11+ |
| LLM | Azure OpenAI (GPT-4o, ChatGPT-5.2) |
| Agent Framework | OpenAI Agents SDK |
| API | FastAPI |
| Queue | Redis |
| Deployment | Kubernetes + Flux |

## Development Commands

```bash
# (Commands TBD after implementation begins)

# Run tests
uv run pytest

# Run linter
uv run ruff check .

# Start development server
uv run uvicorn wizard.api:app --reload
```

## Key Concepts

### Skills (Agent Skills Standard)
WIZARD uses the **Agent Skills open standard** ([agentskills.io](https://agentskills.io)) for skill definitions. This enables cross-platform portability with Claude Code, OpenAI Codex, Cursor, and GitHub Copilot.

Each skill contains:
- `SKILL.md` - Frontmatter (metadata) + markdown body (instructions)
- `scripts/` - Validation and helper scripts
- `references/` - Documentation, examples for few-shot learning
- `assets/` - Templates (Jinja2, YAML starters)

WIZARD builds the **Skill Integration Layer** that bridges Agent Skills → OpenAI Agents SDK (not provided by the SDK natively).

### AGENT.md Pattern
Each target repository should have an `AGENT.md` file that teaches WIZARD:
- Repository purpose and structure
- Patterns for common operations
- Standards and constraints
- Validation commands

### Entry Adapters

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brendanbecker) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
