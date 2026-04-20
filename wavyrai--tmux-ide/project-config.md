---
trigger: always_on
description: A CLI tool that turns any project into a tmux-powered terminal IDE using a simple `ide.yml` config file.
---

# tmux-ide

A CLI tool that turns any project into a tmux-powered terminal IDE using a simple `ide.yml` config file.

## Quick Start

```bash
tmux-ide              # Launch IDE from ide.yml
tmux-ide init         # Scaffold ide.yml (auto-detects stack)
tmux-ide inspect      # Show resolved config + live tmux state
tmux-ide stop         # Kill session
tmux-ide attach       # Reattach to running session
```

## ide.yml Format

```yaml
name: project-name # tmux session name

before: pnpm install # optional pre-launch hook

rows:
  - size: 70% # row height percentage
    panes:
      - title: Claude 1 # pane border label
        command: claude # command to run (optional)
        size: 50% # pane width percentage (optional)
        dir: apps/web # per-pane working directory (optional)
        focus: true # initial focus (optional)
        env: # environment variables (optional)
          PORT: 3000

  - panes:
      - title: Dev Server
        command: pnpm dev
      - title: Shell

team: # optional agent team config
  name: my-team

theme: # optional color overrides
  accent: colour75
  border: colour238
  bg: colour235
  fg: colour248
```

### Agent Team Pane Fields

```yaml
panes:
  - title: Lead
    command: claude
    role: lead # optional layout metadata: "lead" or "teammate"
    focus: true
  - title: Frontend
    command: claude
    role: teammate
    task: "Work on components" # suggested task text for your prompts
```

### Orchestrator Config

```yaml
orchestrator:
  enabled: true
  auto_dispatch: true # auto-assign tasks to idle agents
  dispatch_mode: tasks # "tasks" or "goals"
  poll_interval: 5000 # ms between ticks
  stall_timeout: 300000 # ms before nudging idle agent
  max_concurrent_agents: 10
  worktree_root: .worktrees/ # git worktree per task
  master_pane: Master # lead pane excluded from dispatch
  before_run: pnpm install # hook before task starts
  after_run: pnpm lint # hook after task completes
  cleanup_on_done: false # remove worktree after completion
  webhooks: # fire-and-forget event notifications
    - url: https://example.com/hook
      events: [completion, dispatch] # filter (omit = all events)
      secret: my-signing-key # HMAC-SHA256 via X-Signature-256
```

### Widget Pane Types

```yaml
panes:
  - title: War Room
    type: warroom # explorer | changes | preview | tasks | warroom | costs | config
  - title: Explorer
    type: explorer
    target: src/ # optional target path
```

## Architecture

The project is written in TypeScript. Source lives in `src/`, compiled output in `dist/`. Tests run via Node's `--experimental-strip-types`; the published package ships compiled JS from `tsc`.

### Core CLI

- `bin/cli.js` — CLI entry point and top-level error boundary (stays JS, imports from `dist/`)
- `src/launch.ts` — Launch orchestration for tmux sessions
- `src/restart.ts` — Stop + relaunch flow
- `src/init.ts` — Scaffolds ide.yml with smart detection
- `src/stop.ts` — Kills the tmux session
- `src/attach.ts` — Reattach to running session
- `src/send.ts` — Send messages to panes by name/title/role/ID
- `src/orchestrator-status.ts` — Orchestrator status display (agents, tasks, events)
- `src/config.ts` — Programmatic config mutations
- `src/task.ts` — Mission/goal/task CRUD commands
- `src/status.ts`, `src/inspect.ts`, `src/validate.ts`, `src/detect.ts`, `src/ls.ts`, `src/doctor.ts`

### Daemon & Process Lifecycle

- `src/lib/daemon.ts` — Unified background process: pane monitor + orchestrator + command-center HTTP server. Entry: `node dist/lib/daemon.js <session> [port]`
- `src/lib/daemon-watchdog.ts` — Crash recovery wrapper: respawns daemon on crash with exponential backoff (1s→30s cap, 5 crashes/60s limit). Zero business imports.
- `src/lib/session-monitor.ts` — Pure helper functions (computePortPanes, computeAgentStates) used by daemon.ts

### Orchestrator & Task System

- `src/lib/orchestrator.ts` — Autonomous task dispatch engine (dispatch, stall detection, completion, retry, reconciliation, hot reload)
- `src/lib/task-store.ts` — Mission/goal/task CRUD with JSON file persistence in `.tasks/` (atomic writes via temp+rename)
- `src/lib/event-log.ts` — Append-only event log with structured events and webhook integration
- `src/lib/webhook.ts` — Fire-and-forget webhook dispatcher with HMAC signing
- `src/lib/token-tracker.ts` — Agent time/cost accounting
- `src/lib/worktree.ts` — Git worktree creation per task (with orphan cleanup on startup)
- `src/lib/github-pr.ts` — Auto-PR creation on task completion

### Schemas

- `src/schemas/ide-config.ts` — Zod schemas for ide.yml (IdeConfig, Row, Pane, OrchestratorYamlConfig, WebhookConfig)
- `src/schemas/domain.ts` — Zod schemas for tasks, goals, events, panes, agent details

### Command Center (REST API + SSE + WebSocket)

- `src/command-center/server.ts` — Hono REST API with SSE event streaming
- `src/command-center/discovery.ts` — Session discovery, project detail, orchestrator snapshots
- `src/command-center/pane-mirror.ts` — WebSocket terminal mirroring (raw ANSI)
- `src/command-center/schemas.ts` — Request validation schemas

### Widgets (OpenTUI/Solid TUI)

- `src/widgets/resolve.ts` — Widget type → entry point resolution

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wavyrai/tmux-ide](https://github.com/wavyrai/tmux-ide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
