---
trigger: always_on
description: **Analysis Date:** 2026-05-09
---

# Coding Conventions

**Analysis Date:** 2026-05-09

## Naming Patterns

**Files:**
- PascalCase for classes and views: `CalendarService.ts`, `CalendarView.ts`, `SettingsTab.ts`
- PascalCase for interface/type definition files: `types.ts` (in `src/settings/`)
- camelCase for utility files: `pathUtils.ts`, `timezoneUtils.ts`, `viewRenderers.ts`
- SCREAMING_SNAKE_CASE for type declaration files used as ambient types: `ical.d.ts`
- All source files use `.ts` extension

**Classes:**
- PascalCase for all class names: `CalendarService`, `NoteService`, `MemoChron`, `SettingsTab`, `IcsImportService`

**Functions/Methods:**
- camelCase for all methods and functions: `fetchCalendars`, `getEventsForDate`, `renderCalendarGrid`
- Private helper methods are named descriptively with verb prefixes: `buildFilePath`, `generateNoteContent`, `ensureParentFolder`, `collectRecurrenceExceptions`
- Boolean methods use `is`/`has`/`should`/`check` prefixes: `isAllDayEvent`, `hasSourceMismatch`, `shouldSkipEvent`, `isSameDate`, `checkDailyNoteForDate`, `isRemoteUrl`, `isLocalPath`
- Async methods that return nothing meaningful use `void` or omit return type: `async onload()`, `async onunload()`

**Variables:**
- camelCase for local variables and parameters: `enabledSources`, `fetchPromises`, `cacheExpired`
- Destructured parameters prefer names matching the field: `{ periodStart, periodEnd }`

**Constants:**
- Module-level constants use SCREAMING_SNAKE_CASE: `MEMOCHRON_VIEW_TYPE`, `DEFAULT_REFRESH_INTERVAL`, `DEFAULT_NOTE_LOCATION`
- Class-level static readonly constants use SCREAMING_SNAKE_CASE: `NoteService.FRONTMATTER_DELIMITER`, `NoteService.TEAMS_SEPARATOR`, `NoteService.LOCATION_EMOJIS`
- Color palette arrays follow the same SCREAMING_SNAKE_CASE: `CALENDAR_COLOR_PALETTE`

**Types/Interfaces:**
- PascalCase for interfaces: `CalendarEvent`, `CalendarSource`, `MemoChronSettings`, `PathInfo`, `RenderOptions`
- PascalCase for enums: `PathType`
- PascalCase for type aliases: `CalendarViewMode`
- Interfaces for internal-only data structures are non-exported: `CacheData`, `DateElements`, `EventTemplateVariables`
- Exported interfaces representing public API data are in `src/settings/types.ts` or defined alongside their owning class

**Command IDs:**
- kebab-case for Obsidian command IDs: `"force-refresh-calendars"`, `"go-to-today"`, `"toggle-calendar"`

## Code Style

**Formatting:**
- No Prettier or dedicated formatter config detected (no `.prettierrc`, `biome.json`)
- TypeScript compiler enforces style via `tsconfig.json`
- Double quotes for strings in imports: `import { Plugin } from "obsidian"`
- Template literals used for string interpolation: `` `${source.name}` ``
- Trailing commas on multi-line object/array literals (consistent across all files)
- 2-space indentation throughout

**Linting:**
- `@typescript-eslint/eslint-plugin` and `@typescript-eslint/parser` are devDependencies (version 5.29.0)
- No `.eslintrc` or `eslint.config.*` file present — linting rules are not actively enforced via config file
- TypeScript strict options enabled in `tsconfig.json`: `noImplicitAny: true`, `strictNullChecks: true`

## Import Organization

**Order (observed in practice):**
1. Obsidian API imports: `import { Plugin, Notice, TFile } from "obsidian"`
2. Third-party library imports: `import { Component, Event as ICalEvent, parse, Time } from "ical.js"` / `import { DateTime } from "luxon"`
3. Internal settings/types: `import { CalendarSource } from "../settings/types"`
4. Internal main plugin: `import MemoChron from "../main"`
5. Internal utilities (grouped): `import { getPathInfo, isLocalPath } from "../utils/pathUtils"`

No barrel/index files are used. Each module is imported directly by path.

**Path style:** All internal imports use relative paths: `"../services/CalendarService"`, `"./utils/constants"`, `"../settings/types"`

**Path Aliases:** None — `tsconfig.json` sets `baseUrl: "."` but no `paths` aliases are configured.

## Access Modifiers

Private methods are the dominant pattern. Public/protected are used only on lifecycle methods and intentionally exposed API:

```typescript
// Private for all internal helpers
private buildFilePath(event: CalendarEvent): string { ... }
private async ensureParentFolder(filePath: string): Promise<void> { ... }

// Public (no modifier) for Obsidian lifecycle and plugin API
async onload() { ... }
async saveSettings() { ... }
async refreshCalendarView(forceRefresh = false) { ... }

// Private static readonly for class-level constants
private static readonly FRONTMATTER_DELIMITER = "---";

// Static methods for pure utility classes (IcsImportService uses all-static pattern)
static parseSingleEvent(icsContent: string, ...): CalendarEvent { ... }
```

## Error Handling

**Pattern:** `try/catch` with `console.error` + optional `Notice` for user-facing errors. All 27 try blocks have corresponding catch blocks.

**Service layer errors:**
- `CalendarService`: catch at per-calendar and per-fetch level; shows `Notice` for actionable errors (403, 404, CORS, network), logs to console for internal errors
- `NoteService`: catch at method level, uses `console.error` only (no `Notice` to user) — identified gap in CLAUDE.md
- `IcsImportService`: throws `Error` with descriptive messages for caller to handle

**View layer errors:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [formax68/memoChron](https://github.com/formax68/memoChron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
