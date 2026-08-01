---
trigger: always_on
description: The `@robojs/better-stack` plugin integrates Better Stack's uptime monitoring and log ingestion services into Robo.js projects. It provides two core capabilities:
---

# AGENTS - @robojs/better-stack

## 1. Overview & Purpose

The `@robojs/better-stack` plugin integrates Better Stack's uptime monitoring and log ingestion services into Robo.js projects. It provides two core capabilities:

1. **Heartbeat Monitoring**: Periodic pings to Better Stack Uptime for service health monitoring
2. **Logtail Log Drain**: Forwards Robo.js logs to Better Stack's Logtail service with ANSI color fixes

**Installation**:
```bash
npx robo add @robojs/better-stack
```

**Prerequisites**:
- Better Stack account at https://betterstack.com
- Logtail source token (for log integration)
- Heartbeat URL (for uptime monitoring)

**Package Location**: `packages/plugin-better-stack`

**Use Cases**:
- Centralized logging for production Discord bots/activities
- Real-time log search and visualization
- Uptime monitoring with alerting
- Error tracking and analysis
- Performance monitoring via log patterns

## 2. Architecture Overview

### Core Components

**Source Files**:
- `src/core/drain.ts` - LogDrain implementation with ANSI fixes and Logtail integration (79 lines)
- `src/events/_start.ts` - Plugin lifecycle initialization (47 lines)
- `src/events/_stop.ts` - Cleanup heartbeat interval (8 lines)
- `src/events/_restart.ts` - Delegates to _stop.ts (3 lines)
- `src/index.ts` - Public exports (2 lines)
- `config/robo.mjs` - Plugin configuration

### Data Flow

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. Plugin Initialization (_start.ts)                            │
│    ├─ Read config from plugin config or environment variables   │
│    ├─ If sourceToken exists: create Logtail drain               │
│    ├─ Set drain on root logger via logger().setDrain()         │
│    ├─ If heartbeat.url exists: set up interval to ping URL     │
│    └─ Suppress Fetch API experimental warning                   │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 2. Logging Flow (drain.ts)                                      │
│    ├─ User calls logger.info('message')                        │
│    ├─ Drain receives (logger, 'info', 'message')               │
│    ├─ Parse data with inspect() for objects/errors/arrays      │
│    ├─ Apply ANSI fixes (magenta→blue, bold, resets)            │
│    ├─ Write to console (if logger level allows)                │
│    ├─ Forward to Logtail based on level mapping                │
│    └─ Return Promise.all([console, logtail])                   │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 3. Heartbeat Monitoring (_start.ts)                             │
│    ├─ setInterval fires every interval ms (default 5000)       │
│    ├─ Log debug message if debug enabled                       │
│    ├─ fetch(heartbeat.url)                                     │
│    └─ Catch and log errors, continue on failure                │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ 4. Cleanup (_stop.ts, _restart.ts)                              │
│    ├─ Clear heartbeat interval                                 │
│    └─ Prevent memory leaks                                     │
└─────────────────────────────────────────────────────────────────┘
```

### Dependencies

**Runtime Dependencies** (`package.json` lines 32-34):
- `@logtail/node` ^0.5.5 - Logtail SDK for log ingestion

**Peer Dependencies** (`package.json` lines 42-49):
- `robo.js` ^0.10.28 - Required (not optional)

## 3. Better Stack Account Setup Prerequisites

### Account Creation

1. Sign up at https://betterstack.com
2. Free tier available for small projects
3. Paid tiers for higher log volume and retention

### Logtail Source Setup (for log integration)

**Steps**:
1. Navigate to **Logtail** section in Better Stack dashboard
2. Click **Create Source**
3. Select **Node.js** or **Generic** source type
4. Copy the **source token** (long alphanumeric string)
5. Note the **ingesting host** (e.g., `in.logtail.com` for US, `in-eu.logtail.com` for EU)
6. Store token securely (treat as sensitive credential)

**Security**:
- Source token is a secret credential
- Never commit to version control
- Use environment variables in production
- Rotate tokens periodically

### Heartbeat Monitor Setup (for uptime monitoring)

**Steps**:
1. Navigate to **Uptime** section in Better Stack dashboard
2. Click **Create Monitor**
3. Select **Heartbeat** monitor type
4. Name it after your Robo (e.g., "MyBot Heartbeat")
5. Set **Expected Interval** (e.g., 10 seconds - should match your config)
6. Copy the **Heartbeat URL** (e.g., `https://uptime.betterstack.com/api/v1/heartbeat/...`)
7. Configure **Alert Channels** (email, Slack, Discord, etc.)

**Monitor Behavior**:
- Better Stack expects heartbeat within interval
- If no heartbeat received: monitor marked as down
- Alerts sent via configured channels
- Dashboard shows uptime percentage and history


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Wave-Play/robo.js](https://github.com/Wave-Play/robo.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
