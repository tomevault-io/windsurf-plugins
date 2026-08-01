---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# Agents.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kanchi is a Celery task monitoring frontend built with Nuxt.js (Vue 3) that connects to a FastAPI backend via REST API and WebSocket for real-time monitoring of Celery workers and tasks.

## Development Commands

### Quick Start (from project root)
```bash
# Start both backend and frontend
make dev

# View unified logs
make logs
```

### Environment Setup
```bash
npm install
```

### Development
```bash
npm run dev          # Start development server (localhost:3000)
```

### Build & Deployment
```bash
npm run build        # Build for production
npm run generate     # Generate static site
npm run preview      # Preview production build
```

### API Type Generation
```bash
# Generate TypeScript types from backend OpenAPI schema (use the script to avoid formatting issues)
npm run generate:api:local  # For local development (localhost:8765)
npm run generate:api         # For default backend URL

# Or use the script directly
./generate-api-types.sh http://localhost:8765

# ⚠️ IMPORTANT: Always use the axios template to avoid formatting issues
# The default modular template has a bug that generates malformed TypeScript
# The scripts above handle this automatically
```

## Architecture Overview

### Frontend Stack
- **Framework**: Nuxt 4 (Vue 3) with TypeScript
- **Styling**: TailwindCSS with Radix UI components (reka-ui)
- **State Management**: Pinia stores for centralized state
- **Tables**: TanStack Vue Table for data display
- **Real-time**: WebSocket connection with auto-reconnect
- **HTTP Client**: Auto-generated from OpenAPI spec

### Key Architectural Patterns

#### State Management Layer
All application state is managed through Pinia stores:
- `stores/tasks.ts` - Task events, pagination, filtering, stats
- `stores/workers.ts` - Worker status and management  
- `stores/websocket.ts` - WebSocket connection and real-time updates

#### API Service Layer
- `services/apiClient.ts` - Centralized API service using auto-generated types
- `app/src/types/` - Auto-generated TypeScript types from backend OpenAPI (DO NOT EDIT)

#### Component Architecture
- **`components/ui/`** - ONLY shadcn/ui components (installed via npx shadcn-vue)
- **`components/common/`** - Custom reusable components (Pill, Tag, Select, TimeInput, IconButton, TimePicker)
- **`components/domain/`** - Domain-specific feature components
  - `tasks/` - Task-related components (TaskCard, TaskDetailSheet, etc.)
  - `workers/` - Worker-related components
  - `orphans/` - Orphan task components
- **`components/layout/`** - Layout components (navbar, etc.)
- Business components consume Pinia stores directly
- All data is type-safe using generated API types

### Real-time Data Flow
1. WebSocket connection managed by `websocket` store
2. Live mode: Events stream directly from WebSocket to UI
3. Static mode: Paginated API calls with manual refresh
4. Stores automatically update components when data changes

## Environment Configuration

### Runtime Config (nuxt.config.ts)
```typescript
runtimeConfig: {
  public: {
    wsUrl: process.env.NUXT_PUBLIC_WS_URL || 'ws://localhost:8765/ws',
    apiUrl: process.env.NUXT_PUBLIC_API_URL || 'http://localhost:8765'
  }
}
```

### Environment Variables
```bash
NUXT_PUBLIC_API_URL=http://localhost:8765    # Backend API URL
NUXT_PUBLIC_WS_URL=ws://localhost:8765/ws    # WebSocket URL
```

## Code Patterns

### Using Stores in Components
```typescript
// Always use stores for data access
const tasksStore = useTasksStore()
const workersStore = useWorkersStore()
const wsStore = useWebSocketStore()

// Reactive data
const { recentEvents, isLoading } = tasksStore

// Actions
await tasksStore.fetchRecentEvents()
tasksStore.setPage(2)
wsStore.setMode('live')
```

### Logging
```typescript
// Use the logger service for unified logging (development mode only)
import { useLogger } from '~/services/logger'

const logger = useLogger()

logger.debug('Debug message', { context: 'data' })
logger.info('Info message')
logger.warning('Warning message')
logger.error('Error message', { error: 'details' })
logger.critical('Critical message')
```

Logs are sent to the backend at `/api/logs/frontend` and written to the unified log file at `agent/kanchi.log`. **This feature only works in development mode** - logs are still written to console but not sent to backend in production.

### Type Safety
- All API calls use auto-generated types from backend OpenAPI
- Import types from `~/services/apiClient` for consistency
- Never manually edit files in `app/src/types/` - they are auto-generated

### Error Handling
- Stores handle errors internally and expose error state
- Components should catch store action errors for user feedback
- Use try/catch around store actions that need user notification

## Component Organization

### Directory Structure
```
components/
├── ui/                    # ONLY shadcn/ui components (installed via CLI)
│   ├── button/
│   ├── input/
│   ├── badge/
│   └── ... (all shadcn components)
│
├── common/                # Custom reusable components
│   ├── Pill.vue          # Status pill component

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getkanchi/kanchi](https://github.com/getkanchi/kanchi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
