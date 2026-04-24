---
trigger: always_on
description: **Aha Loop** is a fully autonomous AI development system that extends [Ralph](https://github.com/snarktank/ralph) with planning, research, and oversight capabilities.
---

# Aha Loop Agent Instructions

## Overview

**Aha Loop** is a fully autonomous AI development system that extends [Ralph](https://github.com/snarktank/ralph) with planning, research, and oversight capabilities.

## Aha Loop System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                     GOD COMMITTEE                                     │
│  Independent oversight layer with supreme authority                  │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                             │
│  │  Alpha  │  │  Beta   │  │  Gamma  │  ← 3 members with full power│
│  └─────────┘  └─────────┘  └─────────┘                             │
│        └──────────┴──────────┘                                      │
│              Council                                                  │
│  awakener.sh │ council.sh │ observer.sh │ powers.sh                 │
└─────────────────────────────────────────────────────────────────────┘
                        │ observe │ control │ intervene
┌─────────────────────────────────────────────────────────────────────┐
│                       USER INTERACTION                               │
│  Vision Builder (interactive) │ Observability Logs │ Maintenance    │
└─────────────────────────────────────────────────────────────────────┘
                                   │
┌─────────────────────────────────────────────────────────────────────┐
│                         ORCHESTRATOR                                 │
│  orchestrator.sh - Full project lifecycle management                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
│  │  Vision  │→ │ Architect│→ │ Roadmap  │→ │ Execute  │           │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘           │
└─────────────────────────────────────────────────────────────────────┘
                                   │
┌─────────────────────────────────────────────────────────────────────┐
│                    PARALLEL EXPLORATION                              │
│  parallel-explorer.sh - Git worktree-based parallel implementation  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                          │
│  │Worktree A│  │Worktree B│  │Worktree C│  → Evaluate → Merge     │
│  └──────────┘  └──────────┘  └──────────┘                          │
└─────────────────────────────────────────────────────────────────────┘
                                   │
┌─────────────────────────────────────────────────────────────────────┐
│                        MAINTENANCE                                   │
│  skill-manager.sh │ doc-cleaner.sh │ resources.sh                   │
└─────────────────────────────────────────────────────────────────────┘
```

### Quick Start

#### Option 1: Interactive Vision Building

```bash
# Start guided vision building
./scripts/aha-loop/orchestrator.sh --build-vision

# Then run full automation
./scripts/aha-loop/orchestrator.sh --tool claude
```

#### Option 2: From Existing Vision

```bash
# Create vision document
cp scripts/aha-loop/templates/project.vision.template.md project.vision.md
# Edit project.vision.md

# Run orchestrator
./scripts/aha-loop/orchestrator.sh --tool claude
```

#### Option 3: Parallel Exploration

```bash
# Explore multiple approaches for a decision
./scripts/aha-loop/orchestrator.sh --explore "authentication strategy"

# Or use directly
./scripts/aha-loop/parallel-explorer.sh explore "caching layer" --approaches "redis,memcached,inmemory"
```

### Directory Structure

```
aha-loop/
├── project.vision.md           # Project goals (user input)
├── project.vision-analysis.md  # Vision analysis (generated)
├── project.architecture.md     # Tech decisions (generated)
├── project.roadmap.json        # Milestones & PRDs (generated)
├── AGENTS.md                   # This file
├── CLAUDE.md                   # Claude-specific instructions
├── .god/                       # God Committee (oversight layer)
│   ├── config.json             # Committee configuration
│   ├── directives.json         # Committee directives to execution layer
│   ├── council/                # Council chamber
│   │   ├── chamber.lock        # Speaking rights lock
│   │   ├── agenda.json         # Current agenda
│   │   ├── minutes/            # Meeting records
│   │   └── decisions/          # Decision records
│   ├── members/                # Member state
│   │   ├── alpha/              # Member Alpha
│   │   ├── beta/               # Member Beta
│   │   └── gamma/              # Member Gamma
│   ├── observation/            # System observations
│   │   ├── system-state.json   # Current state snapshot
│   │   ├── anomalies.json      # Detected anomalies
│   │   └── timeline.json       # Event timeline
│   └── powers/                 # Power usage records
│       ├── interventions.json  # Intervention history
│       └── repairs.json        # Repair history
├── .claude/skills/
│   ├── vision/SKILL.md         # Parse project vision
│   ├── vision-builder/SKILL.md # Interactive vision building
│   ├── architect/SKILL.md      # Design architecture
│   ├── roadmap/SKILL.md        # Plan milestones
│   ├── prd/SKILL.md            # Generate PRDs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YougLin-dev/Aha-Loop](https://github.com/YougLin-dev/Aha-Loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
