---
trigger: always_on
description: Nous (Greek: mind/intellect) is a cognitive agent framework built on Minsky's Society of Mind principles. It gives AI agents persistent memory, decision intelligence, and the ability to learn from experience.
---

# CLAUDE.md - Nous Development Guide

## What is Nous?

Nous (Greek: mind/intellect) is a cognitive agent framework built on Minsky's Society of Mind principles. It gives AI agents persistent memory, decision intelligence, and the ability to learn from experience.

**Status: v0.1.0 shipped and deployed.** All core architecture is live.

## Architecture

```
Cognitive Layer (hooks into LLM calls)
    ├── Brain (decisions, deliberation, calibration, guardrails)
    ├── Heart (episodes, facts, procedures, censors, working memory)
    ├── Context Engine (token budgets, relevance scoring, intent-driven retrieval)
    └── Event Bus (async handlers for automation)

Runtime: Direct Anthropic API + tool dispatch loop
Storage: PostgreSQL + pgvector (one DB, three schemas: brain/heart/system)
API: REST (42 endpoints) + MCP server + Telegram bot (streaming)
```

## Project Structure

```
nous/
├── docker-compose.yml          # Nous agent + Postgres + pgvector
├── Dockerfile                  # Python container with OAT support
├── sql/
│   ├── init.sql                # Base schema (21 tables, 3 schemas)
│   ├── migrations/             # Schema migrations (006-016)
│   └── seed.sql                # Default agent, frames, guardrails
├── nous/                       # Python package (~30,000 lines)
│   ├── config.py               # Settings via pydantic-settings
│   ├── main.py                 # Entry point, component wiring, lifecycle
│   ├── telegram_bot.py         # Telegram interface (streaming + usage)
│   ├── events.py               # Event bus (async pub/sub)
│   ├── utils.py                # Shared utilities
│   ├── storage/                # Database layer (async SQLAlchemy)
│   │   ├── database.py         # Connection pool, session management
│   │   ├── models.py           # ORM models for all 28 tables
│   │   └── migrator.py         # Schema migration runner
│   ├── brain/                  # Decision intelligence organ
│   │   ├── brain.py            # Core: record, query, review, calibrate
│   │   ├── bridge.py           # Structure + function descriptions
│   │   ├── calibration.py      # Brier scores, confidence tracking
│   │   ├── embeddings.py       # pgvector embedding provider
│   │   ├── graph_linker.py     # Cross-type auto-linking (common-template embedding)
│   │   ├── guardrails.py       # CEL expression guardrails
│   │   ├── quality.py          # Decision quality scoring
│   │   ├── schemas.py          # Pydantic models
│   │   └── spreading_activation.py  # Density-gated multi-hop graph traversal
│   ├── heart/                  # Memory system organ
│   │   ├── heart.py            # Core: learn, recall, episode lifecycle
│   │   ├── episodes.py         # Episodic memory
│   │   ├── facts.py            # Semantic memory
│   │   ├── procedures.py       # Procedural memory
│   │   ├── censors.py          # Guardrail censors
│   │   ├── censor_actions.py   # F031: Censor action executor (read-only tools)
│   │   ├── working_memory.py   # Short-term scratch space
│   │   ├── search.py           # Full-text + vector search
│   │   ├── subtasks.py         # Subtask CRUD operations
│   │   ├── schedules.py        # Schedule CRUD operations
│   │   └── schemas.py          # Pydantic models
│   ├── cognitive/              # Cognitive layer (Nous Loop)
│   │   ├── layer.py            # pre_turn / post_turn / end_session
│   │   ├── frames.py           # Frame selection (task, question, decision, etc.)
│   │   ├── context.py          # Token-budgeted context assembly
│   │   ├── deliberation.py     # Pre-action protocol
│   │   ├── intent.py           # Intent classification for retrieval
│   │   ├── dedup.py            # Conversation deduplication
│   │   ├── monitor.py          # Post-turn self-assessment
│   │   ├── usage_tracker.py    # Context usage feedback loop
│   │   └── schemas.py          # TurnContext, TurnResult, etc.
│   ├── handlers/               # Event bus handlers
│   │   ├── episode_summarizer.py  # Episode summary generation
│   │   ├── fact_extractor.py      # Fact extraction from conversations
│   │   ├── knowledge_extractor.py # Pre-prune fact extraction
│   │   ├── decision_reviewer.py   # Automated decision review
│   │   ├── session_monitor.py     # Session timeout monitoring
│   │   ├── sleep_handler.py       # Sleep/reflection handler
│   │   ├── subtask_worker.py      # Async subtask execution
│   │   ├── task_scheduler.py      # Cron/one-shot scheduling
│   │   └── time_parser.py         # Natural language time parsing
│   ├── skills/                 # Skill discovery system (F011)
│   │   ├── parser.py           # SkillParser + SkillManifest
│   │   └── bootstrap.py        # One-time local skill registration
│   ├── heartbeat/              # Proactive monitoring (F034)
│   │   ├── runner.py           # HeartbeatRunner tick loop + triage
│   │   ├── registry.py         # CheckRegistry + BaseCheck ABC
│   │   ├── checks.py           # HealthCheck, SelfInitiatedCheck, EmailCheck
│   │   ├── dynamic.py          # DynamicCheck + DynamicCheckLoader (F034.5)
│   │   └── schemas.py          # Finding, CheckResult, HeartbeatResult

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tfatykhov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
