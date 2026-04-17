---
trigger: always_on
description: This file provides guidance to Claude Code when working with the distributed agent coordination system and PRP methodology in this repository.
---

# CLAUDE.md - Agent Coordination & PRP Framework

This file provides guidance to Claude Code when working with the distributed agent coordination system and PRP methodology in this repository.

## 🎯 Project Nature

This is a **Distributed Agent Coordination Framework** with **PRP (Product Requirement Prompt) Methodology**. The system enables 10 specialized AI agents to work in parallel with minimal context loading, delivering production-ready code through orchestrated collaboration.

## 🤖 Agent System Overview

### The 10 Specialized Agents

1. **PRP Orchestrator** - Coordinates workflow, manages phases, ensures quality gates
2. **Business Analyst** - Requirements, user stories, success metrics, ROI analysis
3. **Context Researcher** - Codebase investigation, documentation research, gotcha identification
4. **Implementation Specialist** - Core development, code implementation, feature building
5. **Validation Engineer** - Testing, quality assurance, acceptance criteria validation
6. **Integration Architect** - System design, API contracts, service integration
7. **Documentation Curator** - Documentation generation, user guides, API docs
8. **Security Auditor** - Security assessment, compliance verification, threat modeling
9. **Performance Optimizer** - Performance tuning, optimization, scalability planning
10. **DevOps Engineer** - Deployment, infrastructure, CI/CD, monitoring

### Optional: Archon MCP Server Enhancement

The framework supports optional integration with [Archon MCP Server](https://github.com/coleam00/Archon) for enhanced capabilities:

- 🧠 **Knowledge Base**: RAG-powered search across PRPs, docs, and code
- 📊 **Visual Dashboard**: Web UI for project and task management
- 👥 **Team Collaboration**: Real-time sync across multiple developers
- 🤖 **AI-Enhanced**: Smart task suggestions and context retrieval
- 🔌 **Optional**: File-based system continues to work without Archon

**See [ARCHON-INTEGRATION.md](ARCHON-INTEGRATION.md) for setup and usage.**

### Agent Coordination Principles

```yaml
coordination:
  task_assignment: "Lock-free claiming via session files"
  context_loading: "Agent-specific views (2-5KB vs 50KB+)"
  communication: "JSON-based registry and sync files"
  handoffs: "Structured protocol with notes and file tracking"
  parallel_work: "Multiple agents on different features simultaneously"
```

## 📁 Core Architecture

### Distributed Workspace Structure

```
.agent-system/              # Agent coordination layer
├── registry/               # Lightweight task management
│   ├── tasks.json         # Central task registry (minimal, efficient)
│   └── dependencies.json  # Task dependency graph
├── sessions/              # Multi-session coordination
│   ├── active/*.lock      # Current session locks
│   └── history/           # Completed sessions
├── agents/{agent}/        # Per-agent workspace
│   ├── context.json       # Agent configuration
│   ├── tasks.json         # Assigned tasks
│   └── changelog.md       # Change history
└── sync/                  # Cross-agent communication
    ├── broadcasts.json    # Important updates all agents see
    ├── handoffs.json      # Task transitions between agents
    └── conflicts.json     # Merge conflict tracking
```

### Context Minimization Strategy

The framework uses a multi-layer approach to minimize context loading:

#### Layer 1: Agent-Specific Views (PRPs/.cache/agent-views/)
```bash
# Generate lightweight views (2-5KB each from 50KB+ PRPs)
python scripts/generate-agent-views.py --all

# Result: 90-95% context reduction
# Business Analyst: Goal, Why, Success Criteria sections only
# Implementation: Technical details, no planning docs
# Validation: Test requirements, no architecture
```

#### Layer 2: Context Loader Configuration (.claude/context-loader.yaml)
```yaml
agents:
  implementation-specialist:
    always_load: [context.json, current_task.json]  # ~2KB
    conditional_load: [task_brief.md, api_contract.md]  # ~3KB
    never_load: [full_prps/*.md, other_agents/logs]  # Saves 45KB+
    max_context_tokens: 8000

  business-analyst:
    always_load: [context.json, tasks.json]
    conditional_load: [requirements.md, user-stories.md]
    max_context_tokens: 8000

  validation-engineer:
    always_load: [context.json, tasks.json]
    conditional_load: [test-requirements.md]
    never_load: [architecture/*.md, planning/*.md]
    max_context_tokens: 8000
```

#### Layer 3: Task-Specific Briefs (PRPs/.cache/task-briefs/)
```bash
# Generate brief for specific task (contains only relevant context)
python scripts/generate-agent-views.py --task TASK-001

# Task brief includes:
# - Task metadata (title, status, priority)
# - Referenced PRP sections only
# - Specific file paths
# - Acceptance criteria
```

**Context Budget**: Maximum 50,000 tokens total across all agents per session
**Per-Agent Budget**: 6,000-12,000 tokens depending on role complexity
**Actual Usage**: 2,000-5,000 tokens after optimization

## 🔄 Workflow Orchestration

### Phase-Based Execution

```mermaid
graph LR
    P0[Init] --> P1[Discovery]
    P1 --> P2[Architecture]
    P2 --> P3[Planning]
    P3 --> P4[Implementation]
    P4 --> P5[Deployment Prep]
    P5 --> P6[Go-Live]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TobiAiHawk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
