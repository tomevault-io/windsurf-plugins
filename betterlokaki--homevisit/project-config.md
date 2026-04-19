---
trigger: always_on
description: **Strictly enforced in this codebase:**
---

# HomeVisit Development Rules for Cursor AI

## SOLID Principles & Code Structure

**Strictly enforced in this codebase:**

- **One type/class per file** - Never define multiple types or classes in a single file
- **Max 100 lines per file** - Split larger files into focused modules
- **Single responsibility** - Each class handles one logic only
- **Helper functions** - When a function needs multiple operations, split into helpers and orchestrate
- **Dependency injection** - Never instantiate dependencies inside classes, inject from outside
- **Interface contracts** - Use interfaces between classes/services, never couple directly
- **Throw first, catch later** - Services throw errors, only catch at the highest layer (controllers/schedulers)
- **Separation of concerns** - Never mix different layers (no business logic in controllers, no data access in services)

## Error Handling: Throw First, Catch Later

**Services and low-level modules must throw errors, never catch them:**

```typescript
// ❌ WRONG - Service catches its own errors
async fetchData(): Promise<Data[]> {
  try {
    const response = await axios.get(url);
    return response.data;
  } catch (error) {
    logger.error("Failed", error);
    return []; // Swallowing the error
  }
}

// ✅ CORRECT - Service throws, let caller decide
async fetchData(): Promise<Data[]> {
  const response = await axios.get(url);
  return response.data;
}
```

**Only catch at the highest layer:**

- **Controllers** - Catch and send HTTP error response
- **Schedulers/Orchestrators** - Catch to allow other operations to continue
- **Server startup** - Catch for graceful degradation or exit

```typescript
// Controller (highest layer) - catches and responds
async getSites(req: Request, res: Response): Promise<void> {
  try {
    const sites = await this.siteService.getAll();
    sendSuccess(res, sites);
  } catch (error) {
    sendError(res, "Failed to fetch sites", 500, error);
  }
}

// Scheduler orchestration - catches per-item for resilience
for (const group of groups) {
  try {
    await this.refreshGroup(group.name);
  } catch (error) {
    logger.error(`Failed for group: ${group.name}`, error);
    // Continue with next group
  }
}
```

## Backend Service Directory Structure

Each service lives in its own directory with co-located interfaces:

```
services/
  serviceName/
    interfaces/
      IServiceName.ts    # Interface definition
      index.ts           # Interface exports
    serviceName.ts       # Implementation
    helperFunction.ts    # Helper functions (if needed)
    index.ts             # Service exports
```

Example: `services/site/interfaces/ISiteService.ts`

## Shared Types Directory Structure

Types in `packages/common` are organized by domain:

```
packages/common/src/models/
  site/       # Site, EnrichedSite, SeenStatus, CoverStatus (formerly UpdatedStatus)
  user/       # User, AuthPayload
  filter/     # SiteFilters, FilterRequest
  enrichment/ # EnrichmentRequestBody, EnrichmentResponseBody
  history/    # SiteHistory, CoverUpdateEntry, MergedHistoryEntry, MergedStatus
  group/      # Group
  api/        # ApiResponse types
  overlay/    # ElasticProviderOverlay, FilterSchema
```

**Rule:** One type per file per subdirectory.

## Backend Dependency Injection

All dependencies injected via constructor. Factory pattern in `controllers/controllerFactory.ts`:

```typescript
const postgrestClient = new PostgRESTClient();
const siteService = new SiteService(postgrestClient);
const groupService = new GroupService(postgrestClient);
export const sitesController = new SitesController(siteService, groupService, ...);
```

## Shared Types

Import from `@homevisit/common` - never duplicate types between apps:

```typescript
import type { EnrichedSite, User, FilterRequest } from "@homevisit/common";
```

## Response Helpers

Use utility functions in `utils/responseHelper.ts`:

```typescript
sendSuccess(res, data);
sendError(res, message, 500, error);
sendValidationError(res, "Missing required parameter");
```

## Frontend Conventions

- **RTL Layout**: All components use `dir="rtl"` - Hebrew is the primary language
- **No px units**: Use `%`, `rem`, or Tailwind relative classes only (never use px)
- **Generic components**: Build components as reusable/generic as possible
- **Context separation**: Split Svelte context into separate files by concern
- **Component drilling**: Pass data via props, use Svelte context for deep sharing
- **Map library**: MapLibre GL for geographic visualization

### Frontend Component Development

When writing any new frontend feature:

1. First create a component for each thing, with drilling properties using best practices
2. Or use context, which is also divided by logic using best practices for frontend
3. **Never use px** - always use `%` and/or relative sizes (we never know which screen will see it)

### Frontend Store Pattern

Svelte stores in `src/stores/visit/` with API clients separated:

- `visitStore.ts` - State management with Svelte writable
- `visitApiClient.ts` - HTTP communication (single responsibility)
- Types separated into individual files

## Architecture Overview

HomeVisit is a monorepo (pnpm workspaces) for aerial photography site visit management with:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/betterlokaki) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
