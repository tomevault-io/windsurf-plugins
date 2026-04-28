---
trigger: always_on
description: This file provides context for Claude Code when working with this repository.
---

# CLAUDE.md

This file provides context for Claude Code when working with this repository.

## Project Overview

AgentOS - A multi-agent demo system built by Agno showcasing 50+ Agno framework features (14 agents, 11 teams, 5 workflows).

## Architecture

```
AgentOS (app/main.py)
├── Agents (14)
│   ├── Docs (agents/docs/)                                      # LLMs.txt documentation agent
│   ├── MCP (agents/mcp/)                                        # External tools via MCP
│   ├── Helpdesk (agents/helpdesk/)                              # HITL + guardrails demo
│   ├── Feedback (agents/feedback/)                              # User feedback + control flow
│   ├── Approvals (agents/approvals/)                            # Approval flows + audit trail
│   ├── Reasoner (agents/reasoner/)                              # Reasoning + multi-model + fallback
│   ├── Reporter (agents/reporter/)                              # Structured output + file generation
│   ├── Contacts (agents/contacts/)                              # Entity memory + relationships
│   ├── Studio (agents/studio/)                                  # Multimodal media (DALL-E, TTS, FAL, Luma)
│   ├── Scheduler (agents/scheduler/)                            # Schedule management (SchedulerTools)
│   ├── Taskboard (agents/taskboard/)                            # Session state + agentic state
│   ├── Compressor (agents/compressor/)                          # Tool result compression
│   ├── Injector (agents/injector/)                              # Dependency injection via RunContext
│   └── Craftsman (agents/craftsman/)                            # Skills system (LocalSkills)
├── Teams (11)
│   ├── Pal (agents/pal/)                                        # Personal knowledge agent (team)
│   ├── Dash (agents/dash/)                                      # Data analyst (team)
│   ├── Coda (agents/coda/)                                      # Coding agent (team)
│   ├── Research Coordinate (teams/research/)                    # Team coordinate mode
│   ├── Research Route (teams/research/)                         # Team route mode
│   ├── Research Broadcast (teams/research/)                     # Team broadcast mode
│   ├── Research Tasks (teams/research/)                         # Team tasks mode
│   ├── Investment Coordinate (teams/investment/)                # Investment team coordinate
│   ├── Investment Route (teams/investment/)                     # Investment team route
│   ├── Investment Broadcast (teams/investment/)                 # Investment team broadcast
│   └── Investment Tasks (teams/investment/)                     # Investment team tasks
└── Workflows (5)
    ├── Morning Brief (workflows/morning_brief/)                 # Daily parallel briefing
    ├── AI Research (workflows/ai_research/)                     # Daily parallel AI research
    ├── Content Pipeline (workflows/content_pipeline/)           # Parallel + loop + condition
    ├── Repo Walkthrough (workflows/repo_walkthrough/)           # Code → script → narrated audio
    └── Support Triage (workflows/support_triage/)               # Router + condition + escalation
```

All agents share:
- PostgreSQL database (pgvector) for persistence
- OpenAI GPT-5.4 model (configured in `app/settings.py`)
- Chat history and context management

## Key Files

| File | Purpose |
|------|---------|
| `app/main.py` | AgentOS entry point, registers all agents, teams, workflows |
| `app/config.yaml` | Quick prompts for each agent |
| `app/settings.py` | Shared MODEL, agent_db, and environment flags |
| `app/registry.py` | Shared tools, models, and database connections |
| `agents/docs/agent.py` | Docs - Agno documentation agent using LLMs.txt tools |
| `agents/mcp/agent.py` | MCP - Agno documentation agent via live MCP tools |
| `agents/helpdesk/agent.py` | Helpdesk - HITL + guardrails (moderation, PII, injection, output) |
| `agents/feedback/agent.py` | Feedback - user feedback + control flow tools |
| `agents/approvals/agent.py` | Approvals - approval flows + audit trail |
| `agents/reasoner/agent.py` | Reasoner - reasoning + multi-model + fallback |
| `agents/reporter/agent.py` | Reporter - structured output + file generation |
| `agents/contacts/agent.py` | Contacts - entity memory + relationships |
| `agents/studio/agent.py` | Studio - multimodal media generation (DALL-E, FAL, ElevenLabs, Luma) |
| `agents/scheduler/agent.py` | Scheduler - schedule management (create, list, enable/disable, delete) |
| `agents/taskboard/agent.py` | Taskboard - session state + agentic state demo |
| `agents/compressor/agent.py` | Compressor - tool result compression with CompressionManager |
| `agents/injector/agent.py` | Injector - dependency injection via RunContext |
| `agents/craftsman/agent.py` | Craftsman - Skills system with LocalSkills loader |
| `agents/pal/team.py` | Pal team (Navigator, Researcher, Compiler, Linter, Syncer) |
| `agents/dash/team.py` | Dash team (Analyst, Engineer) |
| `agents/coda/team.py` | Coda team (Coder, Explorer, Planner, Researcher, Triager) |
| `teams/research/team.py` | Research Team (4 modes: coordinate, route, broadcast, tasks) |
| `teams/investment/team.py` | Investment Team (4 modes, 7 agents, YFinance) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/agno-agi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-16 -->
