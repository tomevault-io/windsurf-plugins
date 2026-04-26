---
trigger: always_on
description: This file provides context and guidelines for Claude Code when working on this codebase.
---

# CLAUDE.md - Development Guidelines for YARNNN

This file provides context and guidelines for Claude Code when working on this codebase.

## Project Overview

YARNNN is an **autonomous agent platform for recurring knowledge work**. Persistent AI agents connect to work platforms (Slack, Notion), run on schedule, learn from feedback, and produce outputs that improve with tenure.

**Architecture**: Next.js frontend → FastAPI backend → Supabase (Postgres) → Claude API. Agents (identity) + Tasks (work units) as core model.

**Key terminology** (ADR-138, ADR-140):
- **Agent** — persistent domain expert (WHO). Three axes: identity (AGENT.md, evolves), capabilities (type registry, fixed), tasks (assigned, come and go). Pre-scaffolded at sign-up.
- **Agent Types** (ADR-140, v4 domain-steward model) — capability bundles: `competitive_intelligence`, `market_research`, `business_development`, `operations`, `marketing_creative` (domain-stewards) + `executive_reporting` (synthesizer) + `slack_bot`, `notion_bot` (platform-bots). Three classes: **domain-steward** (owns one context domain, multi-step reasoning) vs **synthesizer** (cross-domain composition) vs **platform-bot** (mechanical, scoped to one API).
- **Task** — defined work unit (WHAT). Objective, cadence, delivery, success criteria. Lives in `/tasks/{slug}/TASK.md`. Assigned to one agent. Thin `tasks` DB table for scheduling.
- **Agent Run** — a single execution of an agent on behalf of a task, producing draft/final content
- **Orchestrator / TP** — the user-facing conversational agent. Creates tasks, monitors agents, orchestrates multi-agent work. Absorbs PM coordination role.
- **Workfloor** — the shared workspace substrate. `/workspace/` (identity), `/agents/` (team), `/tasks/` (work), `/knowledge/` (corpus).
- **Perception Pipeline** (was: platform sync) — how agents sense the outside world
- **Knowledge Base** (was: platform_content) — the shared content substrate agents reason over

## Core Execution Disciplines

### 0. Before Proposing Architectural Changes

**ALWAYS check existing ADRs first** before suggesting new patterns or comparing against external systems:

```bash
# Search for existing decisions on a topic
ls docs/adr/ | grep -i "<topic>"
# Or search ADR content
grep -r "<keyword>" docs/adr/
```

Key ADRs that define YARNNN's philosophy (not just implementation):
- **ADR-049**: Context freshness model - SUPERSEDED by ADR-072 (accumulation moat thesis)
- **ADR-059**: Simplified context model - current Memory schema (user_memory), inference removal
- **ADR-062**: Platform context architecture - SUPERSEDED by ADR-072 (unified content layer)
- **ADR-063**: Four-layer model (Memory / Activity / Context / Work) - activity_log, working memory injection
- **ADR-067**: Session compaction and continuity - follows Claude Code's model
- **ADR-072**: Unified Content Layer - platform_content with retention-based accumulation, TP execution pipeline
- **ADR-080**: Unified Agent Modes - one agent (chat + headless), mode-gated primitives, supersedes ADR-061 two-path separation
- **ADR-087**: Agent Scoped Context - per-agent instructions + memory, session routing via agent_id
- **ADR-088**: Trigger Dispatch - `dispatch_trigger()` in `api/services/trigger_dispatch.py`, single decision point for schedule/event/signal triggers (Phase 1 implemented). **Partially superseded by ADR-126** — pulse decision is now upstream of dispatch; `dispatch_trigger()` invoked only when pulse decides "generate"
- **ADR-092**: Agent Intelligence & Mode Taxonomy - five modes (`recurring`, `goal`, `reactive`, `proactive`, `coordinator`); signal processing dissolved from L3; `RefreshPlatformContent` extended to headless; coordinator agents replace `signal_emergent` origin (Implemented — signal processing removed, modes active, coordinator pipeline in `proactive_review.py`). **Partially superseded by ADR-126** — proactive self-assessment generalized to all agents via pulse Tier 2; coordinator mode dissolved into PM pulse Tier 3
- **ADR-101**: Agent Intelligence Model - four-layer knowledge model (Skills / Directives / Memory / Feedback); learned preferences from edit history injected into headless system prompt; `get_past_versions_context()` includes delivered runs
- **ADR-102**: yarnnn Content Platform - agent outputs written as `platform_content` rows with `platform="yarnnn"`, closing the accumulation loop; always retained; searchable by TP and headless agents; no OAuth, no sync
- **ADR-103**: Agentic Framework Reframe - terminology migration from "deliverable" to "agent" throughout codebase. Agents are persistent autonomous entities, not document generators.
- **ADR-104**: Agent Instructions as Unified Targeting - `agent_instructions` is the single targeting layer; dual-injected into system prompt (behavioral constraints) and user message (priority lens); dead infrastructure deleted (DataSource.scope/filters, SECTION_TEMPLATES, unused type_config fields, template_structure)
- **ADR-105**: Instructions to Chat Surface Migration - directives (instructions, audience) flow through chat; configuration (schedule, sources) stays in drawer; design principle in `docs/design/SURFACE-ACTION-MAPPING.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kvkthecreator) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
