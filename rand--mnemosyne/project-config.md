---
trigger: always_on
description: > **Core Principle**: These guidelines establish workflows, decision trees, and checkpoints that protect against wasted time and technical debt. The Work Plan Protocol takes priority for all development work.
---

# Claude Development Guidelines

> **Core Principle**: These guidelines establish workflows, decision trees, and checkpoints that protect against wasted time and technical debt. The Work Plan Protocol takes priority for all development work.

> **Multi-Agent Architecture**: A sophisticated multi-agent system operates continuously in Claude Code to orchestrate work, optimize context, validate quality, and execute tasks in parallel.

> **⚡ AUTOMATIC MEMORY ENFORCEMENT**: Mnemosyne usage is **technically enforced** via hooks. You **cannot bypass** this system:
> - **SessionStart**: Auto-loads memories at every session start
> - **PostToolUse**: Tracks "memory debt" after Edit/Write/Commit
> - **PreToolUse**: **BLOCKS** git push and PR creation if debt > 0
> - **Stop**: Validates memory usage before finishing responses
>
> **Memory debt system**: Every Edit, Write, or Commit increments debt. After 3 events, you'll be prompted. `git push` is **blocked** until memories are stored. **This is not optional.**

## Table of Contents
1. [Multi-Agent Orchestration System](#1-multi-agent-orchestration-system)
2. [Work Plan Protocol](#2-work-plan-protocol) **START HERE**
3. [Agentic Workflow (Beads)](#3-agentic-workflow-beads)
4. [Critical Thinking & Pushback](#4-critical-thinking--pushback)
5. [Language Stack & Tooling](#5-language-stack--tooling)
6. [Testing & Validation](#6-testing--validation)
7. [Version Control & Git](#7-version-control--git)
8. [Experience Design + Development](#8-experience-design--development)
9. [Skills System](#9-skills-system)
10. [Anti-Patterns](#10-anti-patterns)
11. [Quick Reference](#11-quick-reference)

---

## 1. Multi-Agent Orchestration System

Four primary agents operate continuously in every Claude Code session:

### Agent 1: Orchestrator
**Role**: Central coordinator and state manager

**Responsibilities**:
- Coordinate handoffs between agents with zero-copy data passing
- Monitor execution state across parallel workstreams
- Prevent race conditions and deadlocks through dependency-aware scheduling
- Preserve context before compaction (75% threshold → `.claude/context-snapshots/`)
- Maintain global work graph and schedule parallel work

**Context Preservation**: Trigger pre-emptive snapshots at 75% utilization, compress non-critical data, checkpoint before phase transitions.

**Deadlock Prevention**: Maintain dependency graph, detect circular dependencies, implement timeout monitoring, use priority-based preemption.

---

### Agent 2: Optimizer
**Role**: Context and resource optimization specialist

**Responsibilities**:
- Construct optimal context payloads for each agent
- Apply ACE principles: incremental updates, structured accumulation, strategy preservation
- Monitor all context sources: agents, files, commits, plans, beads, skills, session
- Prevent brevity bias and context collapse
- **Dynamically discover and load relevant skills from filesystem**

**Skills Discovery**:
```
Task → Analyze keywords/domains
     → Scan skills/ directory
     → Score relevance (0-100)
     → Load top 3-7 skills
     → Cache for session
     → Refine as needs evolve
```

**Context Budget Allocation**: Critical (40%), Skills (30%), Project (20%), General (10%)

---

### Agent 3: Reviewer
**Role**: Quality assurance and validation specialist

**Responsibilities**:
- Validate intent satisfaction, documentation, test coverage
- Fact-check claims, references, external dependencies
- Check for anti-patterns and technical debt
- Block work until quality standards met
- Mark "COMPLETE" only when all gates pass

**Quality Gates** (all must pass):
- [ ] Intent satisfied
- [ ] Tests written and passing
- [ ] Documentation complete
- [ ] No anti-patterns
- [ ] Facts/references verified
- [ ] Constraints maintained
- [ ] No TODO/mock/stub comments

---

### Agent 4: Executor
**Role**: Primary work agent and sub-agent manager

**Responsibilities**:
- Follow Work Plan Protocol (Phases 1-4)
- Execute atomic tasks from plans
- Spawn sub-agents for safe parallel work
- Apply loaded skills
- Challenge vague requirements
- Implement code, tests, documentation
- Commit at checkpoints

**Sub-Agent Spawning** (all must pass):
- [ ] Task truly independent
- [ ] Context budget allows
- [ ] No circular dependencies
- [ ] Clear success criteria
- [ ] Handoff protocol established
- [ ] Rollback strategy exists

---

## 2. Work Plan Protocol

**MANDATORY**: Follow four phases for ALL work. Do not skip phases.

### Phase 1: Prompt → Spec
**Goal**: Transform request into clear specification

**Process**:
1. READ user request
2. DISCOVER relevant skills (Optimizer scans skills/)
3. LOAD discovered skills
4. IDENTIFY ambiguities
5. ASK clarifying questions
6. CONFIRM tech stack, deployment, constraints
7. WRITE spec.md
8. SUBMIT to Reviewer

**Exit Criteria**: Intent clear, ambiguities resolved, tech stack confirmed, skills loaded, spec reviewed.

---

### Phase 2: Spec → Full Spec
**Goal**: Decompose into components with dependencies and test plan

**Process**:
1. DECOMPOSE into components
2. IDENTIFY dependencies
3. DEFINE typed holes (interfaces/contracts)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rand/mnemosyne](https://github.com/rand/mnemosyne) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
