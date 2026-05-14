---
trigger: always_on
description: TypeScript wrapper for Sunsama's GraphQL API. Provides type-safe access to daily planning and task management functionality. Published as NPM package with dual CJS/ESM builds.
---

# Sunsama API TypeScript Wrapper - LLM Context

## Project Overview
TypeScript wrapper for Sunsama's GraphQL API. Provides type-safe access to daily planning and task management functionality. Published as NPM package with dual CJS/ESM builds.

## Technical Stack
- **Language**: TypeScript (strict mode)
- **Runtime**: Node.js >= 16
- **Build**: Multiple targets (CJS, ESM, TypeScript declarations)
- **Testing**: Vitest with unit and integration tests
- **Package Manager**: pnpm (development), npm (distribution)
- **GraphQL**: gql-tag for query definitions
- **Special Dependencies**: Yjs (collaborative editing), zod (validation)

## Development Commands
```bash
pnpm build          # Build all targets (CJS, ESM, types)
pnpm test           # Unit tests only (fast, mocked)
pnpm test:integration  # Integration tests (real API, requires .env)
pnpm typecheck      # Type checking without build
pnpm lint           # ESLint
pnpm format         # Prettier
npx changeset       # Create version bump changeset
pnpm release        # Publish to npm (runs build + test + lint first)
```

## Project Structure
```
src/
├── client/                    # SunsamaClient (assembled via inheritance chain)
│   ├── index.ts               # SunsamaClient = final assembled class (thin)
│   ├── base.ts                # SunsamaClientBase — auth, HTTP, session, generateTaskId
│   └── methods/               # Domain method classes (each extends the previous)
│       ├── user.ts            # getUser, getUserTimezone, getStreamsByGroupId, task queries
│       ├── task-lifecycle.ts  # createTask, deleteTask, updateTaskComplete/Uncomplete
│       ├── task-updates.ts    # updateTaskText/Notes/PlannedTime/DueDate/Stream/SnoozeDate
│       ├── subtasks.ts        # createSubtasks, updateSubtaskTitle, addSubtask, etc.
│       ├── task-scheduling.ts # reorderTask
│       └── calendar-events.ts # createCalendarEvent, updateCalendarEvent
├── queries/                   # GraphQL queries/mutations by domain
│   ├── tasks/                 # Task operations
│   ├── streams/               # Stream operations
│   ├── user/                  # User operations
│   └── fragments/             # Shared GraphQL fragments
├── types/                     # TypeScript type definitions
├── utils/                     # Utility functions
│   ├── collab.ts              # Yjs snapshot helpers (createCollabSnapshot, etc.)
│   ├── conversion.ts          # HTML ↔ Markdown conversion
│   ├── validation.ts          # Zod schemas
│   └── dates.ts               # Timezone/date utilities
├── errors/                    # Custom error classes
└── __tests__/
    ├── integration/           # Real API tests (shared auth, auto cleanup)
    └── *.test.ts              # Unit tests (mocked)
```

**Client inheritance chain** (bottom to top):
```
SunsamaClientBase → UserMethods → TaskLifecycleMethods → TaskUpdateMethods → SubtaskMethods → TaskSchedulingMethods → CalendarEventMethods → SunsamaClient
```

## Architecture Patterns

### GraphQL Client
- Uses native `fetch` API
- Cookie-based authentication
- GraphQL mutations for all operations
- Type-safe query/mutation functions with gql-tag

### Error Handling
- Custom error hierarchy: `SunsamaError` → `SunsamaAuthError` / `SunsamaApiError` / `SunsamaValidationError`
- Always throw errors, never return error objects
- Include context in error messages
- Use the right subclass:
  - `SunsamaAuthError` — authentication failures only (login failed, session expired, cannot determine user ID)
  - `SunsamaApiError` — API-level failures with HTTP status (use the existing throw sites in `graphqlRequest` as the primary source)
  - `SunsamaValidationError` — invalid caller input (bad format, out-of-range values); accepts optional `field` parameter
  - `SunsamaError` — generic errors that don't fit a more specific class (e.g. missing response data)
  - **Never** use `SunsamaAuthError` for validation errors or data-not-found conditions

### Type Safety
- Strict TypeScript configuration
- Zod schemas for runtime validation
- No `any` types allowed
- Discriminated unions for variant types (e.g., task integration types)

### Collaborative Editing (Yjs)
**CRITICAL**: Task notes use Yjs for real-time sync with Sunsama UI.

Required structure:
```typescript
Y.XmlFragment('default')
  └─ Y.XmlElement('paragraph')
      └─ Y.XmlText
          └─ actual content
```

**NOT** `Y.Text` directly - this breaks Sunsama UI sync.

Functions (in `src/utils/collab.ts`, exported from `src/utils/index.ts`):
- `createCollabSnapshot(taskId, notes)` - Create initial snapshot
- `createUpdatedCollabSnapshot(existingSnapshot, newContent)` - Update existing

## Testing Patterns

### Unit Tests
- Location: `src/__tests__/**/*.test.ts` (excluding `integration/`)
- Use Vitest with mocked dependencies
- No real API calls
- Fast, run in CI/CD

### Integration Tests
- Location: `src/__tests__/integration/*.test.ts`
- Real API calls with credentials from `.env`
- **MUST use shared authentication pattern** to avoid rate limiting

**Pattern for new integration tests:**
```typescript
import { getAuthenticatedClient, hasCredentials, trackTaskForCleanup } from './setup.js';


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robertn702/sunsama-api](https://github.com/robertn702/sunsama-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
