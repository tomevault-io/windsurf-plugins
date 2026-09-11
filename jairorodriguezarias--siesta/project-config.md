---
trigger: always_on
description: This document describes the autonomous agent system that powers Siesta: the roles, how they interact, the skills they use, and the knowledge base that connects them.
---

# AGENTS.md — Siesta Agent System

This document describes the autonomous agent system that powers Siesta: the roles, how they interact, the skills they use, and the knowledge base that connects them.

---

## Overview

Siesta uses a **dual-model architecture**: GLM 5.2 (via `pi`, Ollama Cloud) plays the roles that must hold the text protocol — planner, consultant, human-proxy — while Gemma4 31B (Ollama Cloud) is the worker that writes, reviews and verifies code. A pipeline orchestrator (`python3 -m pipeline`) coordinates them across 7 phases, with per-issue context loading, post-issue logging, and per-issue learning. The local Ollama daemon acts as the proxy to Ollama Cloud; since round-9 all roles route to cloud models (the local 8B worker's 8K served window was the pomodoro run's bottleneck).

```
┌─────────────────────────────────────────────────────────────┐
│                  python3 -m pipeline                         │
│                   (Orchestrator - Phase 0-7)                  │
│                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌───────────────┐  │
│  │  GLM 5.2     │    │  Gemma 4     │    │  GLM 5.2      │  │
│  │  (Planner)   │    │  (Worker)    │    │  (Consultant) │  │
│  │              │    │              │    │               │  │
│  │ • Interview  │    │ • Execute    │    │ • Resolve     │  │
│  │ • Spec       │    │   issues     │───→│   doubts      │  │
│  │ • Plan       │    │ • Review     │    │ • Deep        │  │
│  │ • Proxy      │    • • Verify     │    │   diagnosis   │  │
│  └──────────────┘    └──────────────┘    └───────────────┘  │
│         │                   │                   │           │
│         └───────────────────┼───────────────────┘           │
│                             ▼                               │
│                    ┌──────────────┐                         │
│                    │  KB Graph    │                         │
│                    │  (JSON)      │                         │
│                    │  per-project │                         │
│                    │  + global    │                         │
│                    └──────────────┘                         │
│                             │                               │
│                             ▼                               │
│                    ┌──────────────┐                         │
│                    │  Learner     │                         │
│                    │  (GLM 5.2)   │                         │
│                    │              │                         │
│                    │ • Per-issue  │                         │
│                    │   learning   │                         │
│                    │ • Skill      │                         │
│                    │   updates    │                         │
│                    └──────────────┘                         │
└─────────────────────────────────────────────────────────────┘
```

---

## Agent Roles

### 1. Planner — GLM 5.2 (via pi)

**When:** Phases 0, 1, 2

**Responsibilities:**
- **Phase 0 (Interview):** Asks the human one question at a time until ~95% confidence about what to build. When confident, outputs `INTENT_FINALIZED:`. The human then leaves.
- **Phase 1 (Spec):** Autonomously writes `spec.md` with: project name, tech stack, structure, features, acceptance criteria, testing approach, boundaries. No questions — decides alone.
- **Phase 2 (Plan):** Reads the spec and writes `issues.md` with ordered, atomic issues. Each issue has: title, description, acceptance criteria, dependencies.

**Skills used:**
- `interview-me` (Phase 0)
- `spec-driven-development` (Phase 1)
- `planning-and-task-breakdown` (Phase 2)

**KB interaction:** Loads standing architectural principles from the global KB before writing the spec (they are mandatory for every project). Logs the human intent as a node, then the spec as a node, then each issue as a node, with `parent_of` edges linking them.

---

### 2. Worker — Gemma4 31B (cloud, since round-9; was local 8B)

**When:** Phase 3 (Execute), Phase 4 (Review), Phase 5 (Verify)

**Responsibilities:**
- **Phase 3:** Executes each issue following TDD (Red → Green → Refactor). Writes code and tests. If stuck, outputs `CONSULT:` with a specific question, context, and code. If a skill says "ask the human", outputs `PROXY_REQUEST:`.
- **Phase 4:** Reviews all code across 5 axes: correctness, readability, architecture, security, performance. Outputs `REVIEW_PASSED:` or `REVIEW_FAILED:`.
- **Phase 5:** Verifies the project runs locally. Detects project type (incl. packages with `__main__.py`, run as `python -m <pkg>`), tries to run it, fixes if needed. Persists the verdict to `verify_verdict.txt` — phase 6 records decision+commit or blocker+`UNVERIFIED` commit per the real verdict.

**Skills used:**
- `incremental-implementation` (Phase 3)
- `test-driven-development` (Phase 3)
- `debugging-and-error-recovery` (Phase 3, 5)
- `issue-executor` (Phase 3 — factory custom)
- `code-review-and-quality` (Phase 4)
- `code-simplification` (Phase 4)

**Stuck protocol:**
```
CONSULT: <specific question>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jairorodriguezarias/siesta](https://github.com/jairorodriguezarias/siesta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
