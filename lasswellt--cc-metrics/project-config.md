---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Code Metrics Dashboard is a Node.js application that monitors Claude Code usage via OpenTelemetry Protocol (OTLP). It receives telemetry data from Claude Code, stores it in RethinkDB, and displays real-time metrics through a web dashboard.

## Essential Commands

### Development
```bash
npm start                # Start the dashboard server
npm run dev             # Same as start (alias)
npm run debug           # Start with DEBUG=true for verbose logging
npm run recalc-stats    # Recalculate aggregated stats from sessions
```

### Database Setup
```bash
npm run setup           # Initialize RethinkDB database schema
                        # (Runs automatically on first server start)
```

**Note:** Database setup happens automatically when you first start the server. The `npm run setup` command is useful for:
- Manual initialization before starting the server
- Troubleshooting database issues
- CI/CD pipelines
- Verifying database connectivity

### Testing
Test the OTLP endpoint manually:
```bash
curl http://localhost:4318/api/debug/metrics              # View raw metrics
curl http://localhost:4318/api/debug/stats-comparison    # Compare session totals vs aggregated stats
```

### Database
```bash
rethinkdb --http-port 8081               # Start RethinkDB server (web UI on port 8081)
docker run -d --name rethinkdb \        # Run RethinkDB in Docker
  -p 28015:28015 -p 8081:8080 rethinkdb
```

## Git Workflow

**CRITICAL: The main branch is protected. Always create a feature branch for your work.**

### Branch Management Rules

1. **Always create a new branch for each session/task**
   ```bash
   git checkout -b feature/descriptive-name
   # Example: feature/add-cost-tracking
   # Example: fix/metrics-aggregation-bug
   ```

2. **Branch naming conventions:**
   - `feature/` - New features or enhancements
   - `fix/` - Bug fixes
   - `refactor/` - Code refactoring
   - `docs/` - Documentation updates
   - `test/` - Test additions or updates

3. **Workflow for each session:**
   ```bash
   # Start of session: Create and switch to new branch
   git checkout -b feature/your-feature-name

   # Make changes, then commit
   git add .
   git commit -m "Descriptive commit message"

   # Push to remote
   git push -u origin feature/your-feature-name

   # Create pull request (when ready)
   gh pr create --title "Feature: Your feature name" --body "Description of changes"
   ```

4. **Never commit directly to main:**
   - Main branch is protected and requires pull requests
   - All changes must go through a feature branch
   - This ensures code review and maintains stability

5. **Keep branches focused:**
   - One branch per logical change or feature
   - Small, focused changes are easier to review
   - Commit often with clear messages

## Architecture

### Data Flow
```
Claude Code (with CLAUDE_CODE_ENABLE_TELEMETRY=1)
    ↓ OTLP/HTTP (JSON)
Server.js (Port 4318) - OTLP Receiver
    ↓
Three parallel data paths:
    ├─→ saveMetricToDB()        → RethinkDB metrics table (raw OTLP data)
    ├─→ upsertSession()         → RethinkDB sessions table (per-session aggregates)
    ├─→ updateMetricBucket()    → RethinkDB metric_buckets table (time-bucketed data)
    └─→ updateAggregatedStats() → RethinkDB aggregated_stats table (all-time totals)
    ↓
In-Memory Cache (last 1000 metrics, last 500 events)
    ↓
Dashboard API (Port 3000) + WebSocket for real-time updates
    ↓
Web Dashboard (Chart.js + vanilla JS)
```

### Hybrid Storage Model

The application uses a dual storage strategy:

1. **In-Memory Cache** (`storage` object in server.js:248-318):
   - Last 1000 metrics
   - Last 500 events
   - Current session stats (resets on server restart)
   - Daily/weekly stats with automatic reset
   - Usage limits configuration

2. **RethinkDB Persistent Storage**:
   - `metrics` table: All raw OTLP metric data points
   - `events` table: Event logs (user prompts, tool results, errors)
   - `sessions` table: Per-session aggregated statistics (indexed by sessionId)
   - `metric_buckets` table: Time-bucketed metrics for historical queries
   - `aggregated_stats` table: Single record ('current') with all-time totals

### Database Schema

**Important:** The `sessions` table is the source of truth for historical aggregates. On server startup, `recalculateAggregatedStatsFromSessions()` rebuilds the `aggregated_stats` table from `sessions` data.

### OTLP Metric Names

CRITICAL: Metric names must exactly match between OTLP payloads and database update functions. See FIX-SUMMARY.md for historical issues with metric name mismatches.

Core metric names processed:
- `claude_code.tokens.input`
- `claude_code.tokens.output`
- `claude_code.tokens.cache_read`
- `claude_code.tokens.cache_creation`
- `claude_code.hook.commands_blocked`
- `claude_code.hook.git_failures`
- `claude_code.tool.files_modified`
- `claude_code.tool.calls`
- `claude_code.active_time.cli`
- `claude_code.active_time.planning`
- `claude_code.active_time.user`
- `claude_code.lines_of_code`

### API Endpoints

**OTLP Receivers:**
- POST `/v1/metrics` - Receives OTLP metrics (JSON format)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lasswellt/cc-metrics](https://github.com/lasswellt/cc-metrics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
