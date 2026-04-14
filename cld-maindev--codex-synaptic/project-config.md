---
trigger: always_on
description: The Codex-Synaptic system enhances OpenAI's Codex with advanced multi-agent capabilities, featuring MCP/A2A bridging, neural meshes, swarm coordination, topological constraints, and various consensus mechanisms. This document outlines the comprehensive agent architecture and deployment strategies.
---

# AGENTS.md - Codex-Synaptic Agent System Architecture

## Overview

The Codex-Synaptic system enhances OpenAI's Codex with advanced multi-agent capabilities, featuring MCP/A2A bridging, neural meshes, swarm coordination, topological constraints, and various consensus mechanisms. This document outlines the comprehensive agent architecture and deployment strategies.

## Operator Notes — 2025-11-05

- **Consensus gating:** ✅ **RESOLVED** — Hive-mind runs now automatically deploy 3+ voting agents (consensus_coordinator, review_worker, planning_worker) to ensure RAFT consensus quorum is reached. The quorum threshold has been adjusted to 2 votes minimum (40% quorum factor) for improved availability while maintaining consensus integrity.
- **Autoscaler behaviour:** With the background daemon disabled, idle worker retirement requests cannot execute. Expect scale-down warnings in logs and manually right-size replicas after experiments. See `docs/runbooks/autoscaler-daemon-coordination.md` for operational guidance.
- **Repository hygiene:** Active development is running from the local `codex-synaptic-clone` directory, but upstream pushes must target `github.com/clduab11/codex-synaptic`. Align the folder/remote names before release packaging so automation recipes resolve assets correctly. See `docs/runbooks/workspace-rename-guide.md` for the step-by-step procedure.

## Startup Gate (Codex For macOS)

- Run `codex-synaptic launch --json` before repository work whenever the user asks to launch or verify readiness first.
- Treat launch as a hard gate: if `ok` is `false` (or the command exits non-zero), stop and only return remediation commands.
- Proceed with repository changes only when launch returns `ok=true` and `nextAction="continue"`.
- Default launch gate profiles are `mcp-filesystem`, `mcp-playwright`, and `mcp-desktop-commander`.

## Core Agent Types

### 1. Worker Agents
Base execution units that perform specific computational tasks.

#### CodeWorker
- **Purpose**: Code generation, analysis, and refactoring
- **Capabilities**: Language-specific code generation, syntax analysis, optimization
- **Interface**: Codex API integration, custom code processing pipelines
- **Deployment**: Single instance or clustered for parallel processing

#### DataWorker  
- **Purpose**: Data processing, transformation, and analysis
- **Capabilities**: ETL operations, statistical analysis, ML preprocessing
- **Interface**: Database connectors, file system access, streaming data
- **Deployment**: Distributed across data sources

#### ValidationWorker
- **Purpose**: Code validation, testing, and quality assurance
- **Capabilities**: Static analysis, test generation, security scanning
- **Interface**: CI/CD integration, testing frameworks
- **Deployment**: Pipeline integration, on-demand validation

#### ResearchWorker
- **Purpose**: Repository reconnaissance and intelligence gathering for Tree-of-Thought planning
- **Capabilities**: Contextual research, insight synthesis, knowledge gap detection
- **Interface**: Documentation corpus, memory system, telemetry feeds
- **Deployment**: Paired with CodeWorkers to keep implementation grounded in latest findings

#### ArchitectWorker
- **Purpose**: Design resilient swarm architectures and rollout strategies
- **Capabilities**: Topology assessment, constraint modelling, phased rollout design
- **Interface**: Neural mesh topology, consensus records, swarm coordinator telemetry
- **Deployment**: Activated upfront for major platform upgrades and backlog governance loops

#### KnowledgeWorker
- **Purpose**: Distil swarm artefacts into documentation and broadcast updates
- **Capabilities**: Knowledge synthesis, documentation drafting, communication packaging
- **Interface**: Persistent memory, docs/, operator channels
- **Deployment**: Runs after ReAcT/ToT cycles to keep humans and agents aligned

#### GoalPlanner
- **Purpose**: Drive Goal-Oriented Action Planning (GOAP) workflows for complex, multi-step objectives
- **Capabilities**: A* search-based plan synthesis, precondition/effect tracking, adaptive replanning with OODA feedback loops
- **Interface**: GOAP action manifests under `user-projects/**/goap/`, reasoning planner APIs, consensus manager for approval gates
- **Deployment**: Activated when tasks demand structured decomposition, such as bounty hunts, compliance remediation, or multi-domain rollouts. See the [goal-planner brief](https://gist.github.com/ruvnet/d5d7686dd96d83fc7ef8968cb57d4b57) for the canonical response template.

#### AnalystWorker
- **Purpose**: Generate heatmaps, scorecards, and analytical briefs to prioritise work
- **Capabilities**: Metric synthesis, risk diagnostics, hotspot reporting
- **Interface**: Telemetry streams, workflow artefacts
- **Deployment**: Activated before planning to sharpen backlog focus

#### SecurityWorker
- **Purpose**: Safeguard the platform with static reviews and threat modelling
- **Capabilities**: Security diagnostics, mitigation planning, guardrail definition

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cld-maindev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
