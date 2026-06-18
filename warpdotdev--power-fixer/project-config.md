---
trigger: always_on
description: This file provides guidance to WARP (warp.dev) when working with code in this repository.
---

# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Repository Structure

PowerFixer is split across two repositories:
- **power-fixer** (this repo) - TUI client application
- **power-fixer-server** (`../power-fixer-server/`) - Backend API server

## Build & Run Commands

```bash
# Start the TUI client (requires server to be running)
./script/run                                     # Connects to localhost:3001
./script/run --server http://localhost:3001      # Explicit server URL
./script/run --server https://your-ngrok.domain  # Connect to remote server

# Development commands
cargo check     # Check compilation
cargo fmt       # Format
cargo clippy    # Lint
cargo build --release  # Build only

# To run the server, see ../power-fixer-server/
```

## System Architecture

Power Fixer is a **client-server system** for GitHub issue triage with AI agent support:

### PowerFixer TUI (This Repository)
A ratatui/crossterm TUI application that runs locally. **Key principle: the TUI stores NO persistent state to disk**. All agent state is kept in an **in-memory cache** during the running session. The TUI is purely a view and controller layer.

**In-Memory Cache Layer:**
The TUI maintains a local cache of all agent status information for instant access without network calls. This cache is stored in the `App` struct and includes:
- `cached_agents` - All agents (fix, dedupe, triage) with their status
- `cached_triage_runs` - Power triage batch runs
- `inbox_states` - Read/archived state for inbox items

**Cache Population & Updates:**
1. **Startup**: `initial_agent_sync()` fetches full state from server via `GET /api/v1/state` (retries up to 10 times if server is not ready)
2. **Real-time WebSocket**: Server pushes updates via WebSocket, handled by `handle_agent_event()` in main.rs
3. **Server-side polling**: Server runs a background loop every 5 seconds, polling Warp's REST API for all active tasks and broadcasting changes via WebSocket
4. **Client-side polling (backup)**: `AgentClient` also polls the server periodically as a redundant mechanism
5. **Manual refresh**: User can trigger sync via 's' key in agent view

**Responsibilities:**
- Render the UI from the in-memory cache (instant, no network calls)
- Receive and apply WebSocket updates to the cache
- Make HTTP calls to server only for write operations (launch agent, update inbox state, etc.)
- Directly fetch GitHub issue data via `gh` CLI (exception to the "server-only" rule for read operations)

### PowerFixer Server (Separate Repository)
The server is located at `../power-fixer-server/`. See that repository's WARP.md for server-specific documentation.

**Key server responsibilities:**
- Single source of truth for all agent-related state (Postgres DB)
- WebSocket server for real-time updates to TUI clients
- Background polling of Warp's REST API for task status
- Callback API for cloud agents to report status

## Data Flow

```
┌──────────────────────────────────────┐
│    PowerFixer TUI (this repo)        │
│  ┌────────────────────────────────┐  │
│  │    In-Memory Cache             │  │
│  │  (cached_agents, etc.)         │◄─┼─── UI reads from cache (instant)
│  └────────────────────────────────┘  │
│           │              ▲           │
│     startup sync    WebSocket push   │
│           │              │           │
└───────────┼──────────────┼───────────┘
            │              │
            ▼              │
┌──────────────────────────────────────┐
│  PowerFixer Server (separate repo)   │
│     ../power-fixer-server/           │
│  ┌─────────────┐  ┌───────────────┐  │
│  │ REST API    │  │ WebSocket Srv │  │
│  └─────────────┘  └───────────────┘  │
│                    ...               │
└──────────────────────────────────────┘
```

**Cache Data Flow:**
1. **TUI reads from cache** - All UI rendering reads from `cached_*` fields, never making network calls
2. **Cache miss triggers fetch** - If cache is empty/stale, TUI calls server to populate it
3. **Server pushes to cache** - WebSocket connection receives real-time updates that update the cache
4. **Server polls external APIs** - Server's Warp API Poller and Callback API receive updates, persist to DB, then broadcast via WebSocket

**Startup Flow:**
1. TUI starts, creates empty cache
2. `initial_agent_sync()` calls `GET /api/v1/state` to populate cache
3. TUI establishes WebSocket connection for real-time updates
4. Server broadcasts updates as agents report progress → cache is updated
5. UI renders from cache with zero latency

**Manual Sync:** TUI can call the server at any time to get a fresh state snapshot (e.g., 's' key in agent view), but the normal flow relies on WebSocket pushes keeping the cache current.

## User Flows

See [userflows.md](userflows.md) for comprehensive navigation diagrams and state machine documentation.

### Primary Triage Flow
Go issue-by-issue through untriaged GitHub issues. For each issue:

1. **Skip** - Move to next issue
2. **Triage** - Add comment, label, or send Slack message for a Warper to handle
3. **Assign Agent** - Launch an cloud agent to attempt an automatic fix

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [warpdotdev/power-fixer](https://github.com/warpdotdev/power-fixer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
