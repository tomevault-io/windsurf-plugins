---
trigger: always_on
description: **Do NOT create/update documentation unless explicitly requested.** Only deliver code changes by default.
---

# Project Rules

## Critical Policies

### Documentation Policy

**Do NOT create/update documentation unless explicitly requested.** Only deliver code changes by default.

### Language Policy

All code comments and logs **must be in English**. Convert non-English comments to English when encountered.

### Database Access Policy

**Frontend MUST NOT directly access SQLite.** All database access goes through backend API handlers.

```typescript
// ❌ Wrong: Direct DB access
const db = await Database.load('sqlite:ido.db')

// ✅ Correct: Use API client
import { getActivities } from '@/lib/client/apiClient'
const activities = await getActivities({ limit: 50 })
```

## API Handler Policy

### 1. Never Use `pyInvoke` Directly

Always use auto-generated functions from `@/lib/client/apiClient` for type safety and automatic camelCase↔snake_case conversion.

### 2. Always Define Structured Request/Response Models

**CRITICAL RULE:** All handlers must use Pydantic models inheriting from `backend.models.base.BaseModel`.

**NEVER use `Dict[str, Any]` as return type** - this prevents TypeScript type generation for the frontend.

```python
from backend.handlers import api_handler
from backend.models.base import BaseModel, TimedOperationResponse

# ❌ WRONG - Dict prevents TypeScript type generation
@api_handler()
async def bad_handler() -> Dict[str, Any]:
    return {"success": True, "data": {...}}

# ✅ CORRECT - Pydantic model enables TypeScript type generation
class MyRequest(BaseModel):
    user_input: str

class MyResponse(TimedOperationResponse):
    data: Optional[MyDataModel] = None

@api_handler(body=MyRequest, method="POST", path="/endpoint", tags=["module"])
async def good_handler(body: MyRequest) -> MyResponse:
    return MyResponse(
        success=True,
        data=MyDataModel(...),
        timestamp=datetime.now().isoformat()
    )
```

**Available Base Response Models:**

- `OperationResponse` - Basic response with success/message/error
- `OperationDataResponse` - Adds optional `data` field
- `TimedOperationResponse` - Adds `timestamp` field
- Define custom models in `backend/models/responses.py` for complex data structures

**After adding handlers:**

1. Define response models in `backend/models/responses.py`
2. Import models in your handler file
3. Import handler in `backend/handlers/__init__.py`
4. Run `pnpm setup-backend` to generate TypeScript types
5. Use in frontend: `import { goodHandler } from '@/lib/client/apiClient'`

## Platform Configuration Policy

Platform-specific settings go in platform config files, NOT in `tauri.conf.json`.

**Files (located in `src-tauri/` directory):**

- `src-tauri/tauri.conf.json` - General (cross-platform)
- `src-tauri/tauri.macos.conf.json` - macOS-specific
- `src-tauri/tauri.windows.conf.json` - Windows-specific
- `src-tauri/tauri.bundle.json` - Production builds

**macOS Bundle Targets:**

- `["app"]` - Development (fast, no DMG)
- `["app", "dmg"]` - Production (used by `pnpm bundle`)

**macOS permissions:** Use `src-tauri/Info.plist` for NSAppleEventsUsageDescription, etc.

## Database Guidelines

### SQL Query Management

**All SQL queries go in `backend/core/sqls/queries.py`.** Never inline SQL in handlers.

### Repository Pattern

1. Use repository layer in `backend/core/db/`
2. Expose interfaces through `core/protocols.py`
3. Update `backend/core/sqls/schema.py` for schema changes
4. Use `?` placeholders for parameterized queries
5. Return typed data (TypedDict/Protocol)

### Database Access Workflow

When frontend needs data:

1. Add SQL query to `backend/core/sqls/queries.py`
2. Create repository method in `backend/core/db/`
3. Create API handler in `backend/handlers/`
4. Define Pydantic request/response models
5. Run `pnpm setup-backend`
6. Use auto-generated function in frontend

## Tailwind CSS Guidelines

**Auto-sorted by `prettier-plugin-tailwindcss` via `pnpm format`.**

- Use canonical classes (e.g., `aspect-video` not `aspect-[16/9]`)
- Never use hardcoded colors - use semantic CSS variables from `src/styles/theme-variables.css`
- Examples: `bg-primary`, `text-primary-foreground`, `bg-muted`, `chart-1` through `chart-5`

```typescript
// ❌ Wrong
<div className="bg-blue-500 text-white aspect-[16/9]">

// ✅ Correct
<div className="aspect-video bg-primary text-primary-foreground">
```

## Project Overview

**Tech Stack:** React 19 + TypeScript 5 + Vite 6 + Tailwind CSS 4 + Python 3.14+ (PyTauri 0.8) + Tauri 2.x + SQLite + Zustand 5

**Architecture:** Three-layer (Perception → Processing → Consumption)

- Perception: Keyboard/Mouse/Screenshots
- Processing: Event extraction (LLM) → Activity aggregation (10min cycle)
- Consumption: AI analysis → Task recommendations

**Key Timing:**

- Screenshots: every 0.2s (5/sec/monitor)
- Main loop: every 30s
- LLM trigger: 20+ screenshots (~4s of activity)
- Activity aggregation: every 10 minutes
- Frontend polling: every 30s (incremental updates)

**Data Flow:** RawRecords (60s memory) → Events (LLM) → Activities (10min aggregation) → Tasks (AI-generated)

## Development Commands

```bash
# Setup
pnpm setup              # macOS/Linux
pnpm setup:win          # Windows

# Development
pnpm tauri:dev:gen-ts   # Full app with TS generation (recommended)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UbiquantAI/IDO](https://github.com/UbiquantAI/IDO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
