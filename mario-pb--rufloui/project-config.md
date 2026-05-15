---
trigger: always_on
description: **RuFloUI** is a React 19 web frontend for managing [claude-flow v3](https://github.com/ruvnet/claude-flow) multi-agent orchestration. It wraps the CLI (`npx @claude-flow/cli@latest`) behind an Express + WebSocket backend and presents a full dashboard with real-time agent monitoring, task execution via multi-agent pipelines, and persistent state.
---

# Claude Code Configuration - RuFloUI

## Project Overview

**RuFloUI** is a React 19 web frontend for managing [claude-flow v3](https://github.com/ruvnet/claude-flow) multi-agent orchestration. It wraps the CLI (`npx @claude-flow/cli@latest`) behind an Express + WebSocket backend and presents a full dashboard with real-time agent monitoring, task execution via multi-agent pipelines, and persistent state.

- **Repo**: https://github.com/Mario-PB/rufloui.git
- **Tech Stack**: React 19 + Vite 6 + TypeScript + Express + WebSocket + Zustand
- **Charts**: Recharts
- **Icons**: Lucide React
- **OS**: Windows 11 (use Unix shell syntax in bash commands)

## Quick Start

```bash
npm install
npm run dev          # Starts both frontend (Vite :28588) and backend (Express :28580)
# Or individually:
npm run dev:frontend # Vite dev server on port 28588
npm run dev:backend  # Express API on port 28580 (tsx watch, auto-reloads)
```

The frontend proxies `/api/*` and `/ws` to `localhost:28580` via Vite config.

## Project Structure

```
src/
├── backend/
│   ├── server.ts          # Express API + WebSocket server (~2100 lines)
│   └── jsonl-monitor.ts   # JSONL session file monitor for Agent Viz (~430 lines)
└── frontend/
    ├── main.tsx            # Entry point
    ├── App.tsx             # Router + WebSocket handler + initial data fetch
    ├── api.ts              # API client (fetch wrapper + all endpoint namespaces)
    ├── store.ts            # Zustand global state with sessionStorage persistence
    ├── types.ts            # TypeScript interfaces (Agent, Task, SwarmAgent, etc.)
    ├── styles/
    │   └── global.css      # CSS variables, dark theme, animations
    ├── components/
    │   ├── Layout.tsx       # App shell: sidebar nav (grouped) + header + activity panel
    │   ├── ErrorBoundary.tsx
    │   └── ui/
    │       ├── Button.tsx   # Reusable button with loading/variants
    │       ├── Card.tsx     # Card container with title/actions
    │       └── StatusBadge.tsx # Colored status pill
    └── pages/
        ├── Dashboard.tsx        # System health, agent overview, activity chart
        ├── AgentsPanel.tsx      # Spawn, list, terminate agents
        ├── SwarmPanel.tsx       # Init/shutdown swarm, topology graph
        ├── SwarmMonitorPanel.tsx # Real-time agent cards with live output modal
        ├── AgentVizPanel.tsx    # JSONL tree visualization of Claude sessions
        ├── HiveMindPanel.tsx    # Consensus, broadcast, shared memory
        ├── TasksPanel.tsx       # Kanban task board with live execution output
        ├── SessionsPanel.tsx    # Save/restore sessions
        ├── PerformancePanel.tsx # Benchmarks, latency/throughput charts
        ├── MemoryPanel.tsx      # Store/search/retrieve memories
        ├── HooksPanel.tsx       # Hook configuration
        ├── NeuralPanel.tsx      # Neural network status
        ├── WorkflowsPanel.tsx   # Workflow management with step tracking
        ├── ConfigPanel.tsx      # Configuration editor
        ├── LogsPanel.tsx        # Live activity logs
        └── MemoryGame.tsx       # Easter egg game
```

## Backend Architecture (server.ts)

The backend is a single Express file that wraps CLI commands, manages state, and persists to disk.

### Key Patterns

1. **CLI Wrapper**: `execCli(command, args)` runs `npx @claude-flow/cli@latest <command> <args>` and returns `{ raw, parsed? }`. Supports `--format json` for structured output.

2. **Table Parser**: `parseCliTable(raw)` extracts rows from ASCII tables (`| col1 | col2 |` format) into `Record<string, string>[]`. Column names are lowercased with spaces replaced by underscores.

3. **Persistence Layer** (`.ruflo/state.json`):
   All critical state is persisted to disk as JSON and restored on server startup:
   - `taskStore`, `workflowStore`, `sessionStore` — user-created data
   - `agentRegistry`, `terminatedAgents`, `agentActivity` — agent tracking
   - `swarmConfig` (id, topology, strategy, maxAgents, shutdown flag)
   - `perfHistory`, `lastPerfMetrics`, `benchmarkHasRun` — performance data
   - `currentSwarmAgentIds` — current swarm membership

   Persistence triggers:
   - **Debounced save** (2s) on every `broadcast()` of significant events
   - **Periodic save** every 30s as safety net
   - **Shutdown save** on SIGINT/SIGTERM
   - **Load on startup** via `loadFromDisk()`

4. **Multi-Agent Pipeline** (`launchSwarmPipeline()`):
   When a task is assigned with an active swarm:
   - Phase 1: Coordinator plans subtasks using ALL available agent roles
   - Phase 2: Workers execute in parallel waves respecting dependencies
   - Phase 3: Results synthesized, task marked complete
   Each agent runs as a separate `claude -p` process with `--output-format stream-json`.

5. **Agent Output Buffers** (`agentOutputBuffers: Map`):
   Stores last 500 lines of Claude output per agent. Streamed to frontend via `agent:output` WebSocket events for the live output modal.

6. **In-Memory Stores** (persisted to `.ruflo/state.json`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mario-PB/RuFloUI](https://github.com/Mario-PB/RuFloUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
