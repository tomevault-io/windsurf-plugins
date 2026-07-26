---
trigger: always_on
description: **Activity Timer** is a sophisticated Trello Power-Up that solves time tracking and estimation challenges within Trello boards. It enables team members to track time spent on cards, set estimates, receive notifications when approaching time limits, and export comprehensive reports.
---

# Activity Timer - Trello Power-Up Technical Reference

## Overview

**Activity Timer** is a sophisticated Trello Power-Up that solves time tracking and estimation challenges within Trello boards. It enables team members to track time spent on cards, set estimates, receive notifications when approaching time limits, and export comprehensive reports.

### The Problem It Solves

Traditional time tracking requires context-switching to external tools, leading to:
- Forgotten time entries
- Inaccurate time logs
- Difficulty correlating work with specific tasks
- No visibility into time vs. estimates
- Complex data export processes

Activity Timer integrates time tracking directly into Trello's UI, making it seamless and context-aware.

---

## Architecture & Technology Stack

### Frontend
- **Framework**: Vue 3 (Composition API)
- **Build Tool**: Vite
- **UI Library**: PrimeVue (DataTable, Dropdown, DatePicker, etc.)
- **Styling**: SCSS + PrimeFlex (utility CSS)
- **TypeScript**: Full type safety throughout
- **Error Tracking**: Sentry for production error monitoring
- **Integration**: Trello Power-Up Client API

### Backend Infrastructure (AWS)
- **API Gateway HTTP**: Webhook receiver for Trello events
- **API Gateway WebSocket**: Real-time communication for auto-timer feature
- **Lambda Functions**:
  - HTTP handler for webhook processing
  - WebSocket handler for connection management
- **DynamoDB**: Stores WebSocket connection mappings (connection_id ↔ member_id)
- **CloudFront + S3**: Static asset delivery
- **Infrastructure as Code**: AWS CDK (TypeScript)

### Development Tools
- **Linting**: ESLint + Prettier
- **Type Checking**: vue-tsc
- **Local Dev**: Vite dev server with HTTPS (mkcert)

---

## Data Storage Architecture

### Trello's Plugin Data System

Activity Timer stores **ALL card data** directly in Trello's native storage using the Trello Power-Up API. This is a key architectural decision that provides:
- No external database for card data
- Data persists with the card forever
- Accessible via Trello's REST API
- Automatic sync across all clients

### Storage Keys & Data Structure

All data is stored in **shared** scope (visible to all board members) using compressed array formats to minimize storage usage.

#### 1. Time Ranges (`act-timer-ranges`)
**Purpose**: Completed time tracking sessions
**Structure**: Array of arrays `[memberId, startTimestamp, endTimestamp]`
**Location**: Card-level, shared scope
**Example**:
```typescript
[
  ["5f1a2b3c4d5e6f7a8b9c0d1e", 1633024800, 1633028400],  // 1 hour session
  ["5f1a2b3c4d5e6f7a8b9c0d1e", 1633032000, 1633035600]   // Another session
]
```

#### 2. Active Timers (`act-timer-running`)
**Purpose**: Currently running timers
**Structure**: Array of arrays `[memberId, listId, startTimestamp]`
**Location**: Card-level, shared scope
**Note**: Only one timer per member per board (automatically stops other timers)
**Example**:
```typescript
[
  ["5f1a2b3c4d5e6f7a8b9c0d1e", "5f9a8b7c6d5e4f3a2b1c0d9e", 1633040000]
]
```

#### 3. Estimates (`act-timer-estimates`)
**Purpose**: Time estimates per member
**Structure**: Array of arrays `[memberId, timeInSeconds]`
**Location**: Card-level, shared scope
**Example**:
```typescript
[
  ["5f1a2b3c4d5e6f7a8b9c0d1e", 7200],  // 2 hours
  ["6a2b3c4d5e6f7a8b9c0d1e2f", 3600]   // 1 hour
]
```

#### 4. Auto-Timer Settings (`act-timer-auto-timer`)
**Purpose**: Enable/disable auto-timer feature
**Structure**: `0` or `1`
**Location**: Board-level, shared scope

#### 5. Auto-Timer List ID (`act-timer-auto-timer-list-id`)
**Purpose**: Which list triggers auto-timer start
**Structure**: Trello list ID string
**Location**: Board-level, shared scope

#### 6. Member Settings (private scope)
**Purpose**: Per-member notification thresholds, UI visibility
**Location**: Member-level, private scope
**Note**: Not synced across members

### Data Access Patterns

**Reading Data**:
```typescript
const ranges = await t.get(cardId, 'shared', 'act-timer-ranges', []);
```

**Writing Data**:
```typescript
await t.set(cardId, 'shared', 'act-timer-ranges', serializedRanges);
```

**Storage Limits**: Trello enforces a 4KB limit per storage key. Activity Timer compresses data by:
1. Using minimal array structures (no property names)
2. Unix timestamps instead of ISO strings
3. Member IDs only (names fetched from Trello API when needed)

---

## Key Components & Their Roles

### Core Data Models

#### `Card` (`src/components/card.ts`)
Central orchestrator for all card-related operations.

**Responsibilities**:
- Fetch/calculate time spent (running timers + completed ranges)
- Start/stop time tracking
- Automatically stop timers on other cards when starting a new one
- Handle threshold validation (prevents saving very short trackings)
- Error handling for storage limit exceeded

**Key Methods**:
- `getRanges()`: Fetch completed time ranges
- `getTimers()`: Fetch active timers
- `getEstimates()`: Fetch estimates
- `startTracking(listId)`: Start timer (stops all other timers first)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danniehansen/activity-timer](https://github.com/danniehansen/activity-timer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
