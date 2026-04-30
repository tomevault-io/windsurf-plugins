---
trigger: always_on
description: > **RULE: Update this file whenever code or project structure changes.** Every added, removed, or renamed file, template, config key, or architectural decision must be reflected here before the task is considered done.
---

# CLAUDE.md — Project Rules & Context

> **RULE: Update this file whenever code or project structure changes.** Every added, removed, or renamed file, template, config key, or architectural decision must be reflected here before the task is considered done.

---

## Project Overview

**OpenClaw Multi-Agent Kit** — Production-tested templates for building AI agent teams on [OpenClaw](https://openclaw.sh) with Telegram supergroup integration. This is a template/docs-only repo (no runtime code). It provides SOUL.md personality templates, IDENTITY.md metadata templates, workspace scaffolding, and openclaw.json config snippets for deploying up to 10 autonomous agents coordinated through Telegram topic channels.

**Not a library or app.** Nothing here executes — it's all markdown templates, JSON config examples, and documentation meant to be copied into an OpenClaw workspace.

## Tech Stack / Platform

- **Platform:** OpenClaw (agent orchestration platform)
- **Channel:** Telegram (supergroups with topics)
- **LLM providers:** Anthropic Claude models (sonnet-4-6, opus-4-6, haiku-4-5)
- **Config format:** JSON / JSONC
- **Docs/templates:** Markdown

## Repository Structure

```
.
├── .gitignore                         # OS artifacts and editor files
├── CLAUDE.md                          # This file — project rules & context
├── INSTRUCTIONS.md                    # AI-readable setup guide (8 phases)
├── README.md                          # Project overview & quick start
├── LICENSE                            # MIT
├── docs/
│   ├── agent-design-patterns.md       # How to write effective SOUL.md files
│   ├── scaling.md                     # Scaling guidance: when to add agents, cost, circular triggers
│   ├── supergroup-setup.md            # Step-by-step Telegram supergroup setup (covers multi-bot + native topic routing)
│   ├── skills-system.md               # Native skills system, ClawHub, creating custom skills
│   ├── acpx-telegram.md               # ACPX coding agent backend for Telegram
│   └── telegram-dm-topics.md          # Telegram DM forum topics + ACP binding guide
├── examples/
│   ├── full-team.json                 # Complete 10-agent openclaw.json config
│   └── minimal-team.json              # Minimal 3-agent config (orchestrator + coder + QA)
└── templates/
    ├── openclaw-config.jsonc          # Base config snippet with defaults
    ├── identity/
    │   └── agent-identity.md          # Standard identity template for any agent
    ├── soul/                          # Agent personality templates (SOUL.md)
    │   ├── orchestrator.md            # Lead agent — coordinates all others
    │   ├── coding-agent.md            # Software engineering specialist
    │   ├── qa-agent.md                # Testing and quality assurance
    │   ├── devops-agent.md            # Infrastructure and deployment
    │   ├── research-agent.md          # Market research and intelligence
    │   ├── growth-agent.md            # Analytics and growth experiments
    │   ├── content-agent.md           # Social media content creation
    │   ├── community-agent.md         # Community engagement (Reddit, forums)
    │   ├── leadgen-agent.md           # Prospect research and lead scoring
    │   └── ops-agent.md              # Email, calendar, and data management
    ├── workspace/                     # Shared context file templates
    │   ├── AGENTS.md                  # Orchestrator operations guide
    │   ├── FEEDBACK-LOG.md            # Style corrections and lessons
    │   ├── SIGNALS.md                 # Shared intelligence hub
    │   ├── SUPERGROUP-MAP.md          # Topic and agent mapping
    │   └── THESIS.md                  # Business thesis — north star for all agents
    └── skills/                        # Skill templates (SKILL.md)
        ├── coding-handoff/            # Build→QA→Deploy handoff lifecycle
        ├── research-intel/            # Signal extraction + confidence scoring
        ├── leadgen-qualification/     # ICP scoring + outreach routing
        ├── content-repurpose/         # Cross-channel post repurposing
        ├── ops-triage/                # Priority routing for inbox/calendar/tasks
        ├── telegram-topic-setup/      # Automated topic creation and binding
        └── acpx-session/              # ACPX session management patterns
```

## Architecture — Key Concepts

- **Three Telegram routing models** — Multi-bot routing, native topic routing, and DM forum topics (see docs/)
- **One topic per team** — Teams share a topic channel in a supergroup
- **Primary + Secondary agents** — Primary owns the topic; secondary responds only when @mentioned or triggered
- **Shared context via markdown files** — Agents coordinate through THESIS.md, SIGNALS.md, FEEDBACK-LOG.md (not APIs)
- **Bot-to-bot via `sessions_send`** — Telegram bots cannot see each other's messages; OpenClaw's `sessions_send` bridges them
- **Structured escalation** — Agents escalate to orchestrator; orchestrator escalates to human

### Telegram Routing Models

| Model | Visibility | Best For |
|-------|-----------|----------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raulvidis/openclaw-multi-agent-kit](https://github.com/raulvidis/openclaw-multi-agent-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
