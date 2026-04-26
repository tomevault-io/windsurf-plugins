---
trigger: always_on
description: > "Philosophy must be practical, not theoretical. Specs must be actionable, not ephemeral."
---

# CLAUDE.md — Musonius Development

> "Philosophy must be practical, not theoretical. Specs must be actionable, not ephemeral."
> — Musonius Rufus, 30–101 AD, "The Roman Socrates"

## What Is Musonius

Musonius is an open-source CLI tool that sits **between your intent and your AI coding agents**. It pre-computes codebase context, maintains persistent project memory, and generates optimized handoff documents so any downstream agent (Claude Code, Gemini CLI, Cursor, Aider, Codex) goes straight to surgical execution instead of burning tokens on exploration.

**Musonius is the coach, not the player.** It doesn't write code — it makes every tool that does write code 3–5x more effective.

**Domain:** musonius.dev
**License:** MIT
**Language:** Python 3.12+
**CLI framework:** Click or Typer (prefer Typer for type hints)

---

## Architecture Overview

### Core Components

```
musonius/
├── cli/                    # CLI entry points (Typer)
│   ├── __init__.py
│   ├── main.py             # musonius init | plan | prep | verify | memory
│   └── commands/
│       ├── init.py          # Index codebase, create .musonius/ scaffold
│       ├── plan.py          # Decompose task → phases via scout agent
│       ├── prep.py          # Generate agent-specific context file
│       ├── verify.py        # Cross-model adversarial review
│       └── memory.py        # View/edit project knowledge
├── indexer/                # Core engine — replaces 60% of token-wasting exploration
│   ├── parser.py           # Tree-sitter codebase parser
│   ├── graph.py            # Dependency graph builder
│   └── repomap.py          # Multi-level repo map generator (L0–L3)
├── memory/                 # Persistent project intelligence
│   ├── store.py            # SQLite store for conventions, decisions, failures
│   ├── schema.py           # Memory schema definitions
│   └── query.py            # Memory search and retrieval
├── context/                # The critical output layer
│   ├── generator.py        # Task + index + memory → context document
│   ├── budgets.py          # Token budget allocation and enforcement
│   ├── formats/
│   │   ├── claude.py       # CLAUDE.md generator (XML-style structured)
│   │   ├── gemini.py       # GEMINI.md generator (natural language)
│   │   ├── cursor.py       # .cursorrules generator
│   │   └── generic.py      # AGENTS.md fallback
│   └── serializer.py       # Compact serialization (65% smaller than JSON)
├── router/                 # Cost-aware model routing
│   ├── scout.py            # Free-tier scouting (Gemini Flash)
│   ├── planner.py          # Planning model selection
│   └── verifier.py         # Cross-model review routing
├── mcp/                    # MCP server for universal integration
│   └── server.py           # Exposes musonius_get_plan, _get_context, _verify, _memory_query
├── parallel/               # Git worktree parallel execution (v1.1+)
│   ├── worktree.py         # Worktree lifecycle management
│   ├── coordinator.py      # Phase dependency analysis + parallel dispatch
│   └── merger.py           # Conflict detection and resolution
└── utils/
    ├── tokens.py           # Token counting and budget tracking
    ├── git.py              # Git operations (tags, stash, rollback)
    └── config.py           # Configuration management
```

### Project Data Directory

```
.musonius/
├── constitution.md         # Immutable project rules (auto-generated on init)
├── sot/                    # Source of Truth — architecture decisions, API contracts
│   ├── TECH-001.md         # e.g., "Python 3.12+ with FastAPI"
│   └── API-001.md          # e.g., "All endpoints require OpenAPI docs"
├── epics/                  # Feature specs with requirements + acceptance criteria
│   └── {epic-id}/
│       ├── spec.md
│       └── phases/
│           ├── phase-01.md  # File-level instructions + code context
│           └── phase-02.md
├── memory/
│   ├── conventions.json    # Code patterns and standards
│   ├── decisions.json      # Architecture decisions with rationale
│   ├── failures.json       # Failed approaches (anti-pattern library)
│   └── graph.json          # Dependency graph cache
└── config.yaml             # User preferences, model config, autonomy levels
```

---

## CLI Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `musonius init` | Index codebase, create `.musonius/` scaffold, auto-detect conventions | `musonius init` |
| `musonius plan "task"` | Decompose task into phased plan via scout agent | `musonius plan "add rate limiting to public API"` |
| `musonius prep "task"` | Generate optimized context file for chosen agent | `musonius prep epic-004 --agent claude` |
| `musonius verify` | Capture git diff, cross-model adversarial review | `musonius verify --reviewer gemini` |
| `musonius memory` | View/edit/search project knowledge | `musonius memory search "auth"` |
| `musonius status` | Token usage, phase progress, memory stats | `musonius status` |
| `musonius rollback` | Restore to phase checkpoint | `musonius rollback epic-004 phase-2` |

---

## Key Design Principles

1. **Local-first, always.** Everything runs on the user's machine. No cloud dependency for context. No telemetry. No accounts.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Musonius-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
