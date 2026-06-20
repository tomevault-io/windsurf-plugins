---
trigger: always_on
description: Backend engineering command center for polyglot developers — 8 specialized agents, 15 task categories, language-aware routing, research-first workflow, anti-hallucination guardrails, and automated task completion loops for Python, Rust, TypeScript, Go, Java, C/C++, and Ruby projects on OpenClaw.
---


# PolyForge

> Structured, research-first, language-aware backend engineering workflow for OpenClaw.

## Overview

PolyForge provides a 3-layer agent architecture and opinionated workflow for backend engineers working across Python, Rust, TypeScript, Go, Java, C/C++, and Ruby. Every task flows through triage → planning → execution → verification.

- **3-Layer Architecture**: Strategy (Atlas) → Coordination (Architect, Researcher, Analyst, Critic, Explorer) → Execution (Sprint, Forge)
- **15 Task Categories**: Intent-based routing from quick fixes to deep research to architecture decisions
- **Language-Aware**: Auto-detects Python / Rust / TypeScript / Go / Java / C++ / Ruby, injects relevant skills and verification rules
- **Research-First**: Evaluate technologies before building via structured Tech Briefs
- **Anti-Hallucination Guardrails**: 8 rules injected into every prompt
- **Autorun Loop**: Self-correcting execution with configurable iteration limits
- **Todo Tracking**: Persistent todo management across sessions via `pf_todo_*` tools

## Installation

```bash
git clone https://github.com/DVNghiem/PolyForge polyforge
cd polyforge/plugin
npm install
npm run build
npx pf-setup
```

Add to OpenClaw config:

```json
{
  "plugins": {
    "polyforge": {
      "path": "./polyforge/plugin"
    }
  }
}
```

### Verify Installation

```
/pf status
/pf list
```

## Trigger

This skill activates when:

- User invokes `/triage`, `/research`, `/intake`, `/plan`, `/execute`, or `/work`
- User asks for task routing, technology evaluation, or backend engineering help
- User needs multi-step coding pipeline coordination across Python, Rust, TypeScript, Go, Java, C/C++, or Ruby

## Architecture

### Layer 1: Strategy

| Agent     | Role                                                              | Categories        |
| --------- | ----------------------------------------------------------------- | ----------------- |
| **Atlas** | Orchestrator — triage, plan, delegate, verify, escalate           | apex, all routing |

### Layer 2: Coordination

| Agent          | Role                                                             | Category  |
| -------------- | ---------------------------------------------------------------- | --------- |
| **Architect**  | Read-only architecture consultation — API design, data models    | apex      |
| **Researcher** | Technology evaluation — produces structured Tech Briefs          | research  |
| **Analyst**    | Gap analysis — finds missing requirements before implementation  | review    |
| **Critic**     | Code & plan review — severity-categorized findings, fix guidance | review    |
| **Explorer**   | Codebase search — maps structure, traces dependencies, finds patterns | quick |

### Layer 3: Execution

| Agent      | Role                                                              | Category              |
| ---------- | ----------------------------------------------------------------- | --------------------- |
| **Sprint** | Quick worker — bounded features, bug fixes, docs, TypeScript/Ruby work | quick, typescript, ruby, writing |
| **Forge**  | Deep specialist — complex refactors, async debugging, Rust/Python/Go/Java/C++ | deep, rust, python, go, java, cpp |

### Category-to-Agent Routing

| Category         | Default Agent | Examples                                               |
| ---------------- | ------------- | ------------------------------------------------------ |
| `quick`          | Sprint        | Add a field, fix a typo, rename a function             |
| `deep`           | Forge         | Refactor auth module, implement rate limiter           |
| `apex`           | Architect     | Design a service boundary, evaluate caching strategy   |
| `research`       | Researcher    | Evaluate Axum vs Actix, compare PostgreSQL vs CockroachDB |
| `rust`           | Forge         | Fix lifetime issue, optimize hot path, async trait impl|
| `python`         | Forge         | Add FastAPI endpoint, write async background task      |
| `typescript`     | Sprint        | Add Fastify route, fix type error, write integration test |
| `go`             | Forge         | Add gin handler, fix goroutine leak, write table-driven test |
| `java`           | Forge         | Add Spring Boot endpoint, write JPA repository, fix N+1 |
| `cpp`            | Forge         | Fix memory leak, optimize hot path, write CMakeLists.txt |
| `ruby`           | Sprint        | Add Rails action, write RSpec test, define ActiveRecord scope |
| `review`         | Critic        | Review PR diff, critique a plan, identify security issues |
| `writing`        | Sprint        | Write a tech brief, update API docs, write changelog   |
| `unspecified-low`| Sprint        | Unknown low-complexity tasks                           |

Coding categories (`quick`, `deep`, `rust`, `python`, `typescript`, `go`, `java`, `cpp`, `ruby`) route through `pf_spawn_acp` → `sessions_spawn` for real sub-agent sessions.

## Workflows

### `/work` — Full Engineering Pipeline


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DVNghiem/PolyForge](https://github.com/DVNghiem/PolyForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
