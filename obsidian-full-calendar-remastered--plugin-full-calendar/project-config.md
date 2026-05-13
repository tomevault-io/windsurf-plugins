---
trigger: always_on
description: This plugin integrates FullCalendar.js into Obsidian, supporting local (Full Note, Daily Note) and remote (ICS, CalDAV, Google) calendars. It is TypeScript-based, modular, and designed for robust event management and calendar views.
---


# Full Calendar Plugin for Obsidian

This plugin integrates FullCalendar.js into Obsidian, supporting local (Full Note, Daily Note) and remote (ICS, CalDAV, Google) calendars. It is TypeScript-based, modular, and designed for robust event management and calendar views.

## Essential Architecture

- **UI Layer**: React components (`src/ui/`), FullCalendar.js integration (`src/ui/view.ts`), and view models (e.g., `ViewEnhancer`).
- **Core Layer**: Central event management via `EventCache` (single source of truth) and `EventStore` (in-memory DB, indexes).
- **Calendar Layer**: Pluggable sources (`src/calendars/`): Full Note, Daily Note, ICS, CalDAV.
- **Abstraction Layer**: `ObsidianAdapter` for testable Obsidian API interactions.
- **ChronoAnalyser**: Data visualization (see `src/chrono_analyser/`), consumes `EventCache` via pub/sub for real-time updates.

**Data Flow Example**:
- User actions → EventCache → Calendar implementations → Obsidian vault
- File changes/remote sync → EventCache → UI updates (pub/sub)

## Developer Workflows

- **Bootstrap**:  
	- `npm install` (45s, never cancel)
- **Build/Test**:  
	- `npm run compile` (type check)  
	- `npm run lint` (Prettier)  
	- `npm run test` (Jest, 154 tests)  
	- `npm run build` (esbuild)  
	- `npm run prod` (type check + build)
- **Development**:  
	- `npm run dev` (esbuild watch)  
	- `npm run fix-lint` (auto-format)  
	- `npm run coverage` (test coverage)  
	- `npm run test-update` (update Jest snapshots)
- **Validation**:  
	- Always run `npm run lint && npm run compile && npm run test` before commit
	- Test in `obsidian-dev-vault/` (pre-configured dev vault)
	- Copy `manifest.json` to plugin build directory for Obsidian testing

## Project Conventions

- **Event Storage**:  
	- Full Note: events as separate notes with frontmatter  
	- Daily Note: events as list items with inline metadata
- **Category System**:  
	- Format: `Category - Title` or `Category - Subcategory - Title`
	- Color coding and parsing logic in core
- **Recurring Events**:  
	- Recurrence logic and instance modification supported
- **Internationalization**:  
	- Uses i18next, translation files in `src/features/i18n/locales/`, type-safe keys

## Key Files & Directories

- `src/main.ts` — Plugin entry
- `src/core/EventCache.ts` — Event management
- `src/core/EventStore.ts` — In-memory DB
- `src/calendars/` — Calendar sources
- `src/ui/view.ts` — Calendar view integration
- `src/types/schema.ts` — Zod schemas
- `test_helpers/MockVault.ts` — Obsidian API mocking
- `docs/` — MkDocs documentation

## Integration & Extensibility

- **External dependencies**: FullCalendar.js, React, Luxon (bundled), Obsidian APIs (external)
- **ChronoAnalyser**: Extensible charting via Strategy Pattern; subscribes to EventCache for real-time data
- **Testing**: Unit/integration tests, snapshot updates, coverage enforcement

## Troubleshooting

- Build failures: check TypeScript errors, esbuild config, CSS renaming
- Test failures: run `npm run test-update` for snapshots, check date/timezone issues
- Plugin loading: verify build output, copy manifest, check Obsidian console

## Best Practices

- Minimal, modular changes; follow SOLID/DRY
- Strict formatting (Prettier, Husky hooks)
- Always validate in dev vault before commit
- Reference [Obsidian plugin guidelines](https://docs.obsidian.md/Plugins/Releasing/Plugin+guidelines)


### Repository Structure
```
.
├── README.md
├── CONTRIBUTING.md 
├── package.json           # npm scripts and dependencies
├── esbuild.config.mjs     # build configuration  
├── jest.config.js         # test configuration
├── manifest.json          # Obsidian plugin manifest
├── src/                   # TypeScript source code
│   ├── main.ts           # plugin entry point
│   ├── calendars/        # calendar source implementations
│   ├── core/             # core logic (EventCache, EventStore)
│   ├── ui/               # React components and views
│   └── types/            # TypeScript types and schemas
├── test_helpers/         # test utilities and mocks
├── docs/                 # documentation (MkDocs)
├── tools/                # Python development utilities
└── obsidian-dev-vault/   # development Obsidian vault
```

### Key Source Files
- `src/main.ts` -- Plugin entry point and initialization
- `src/core/EventCache.ts` -- Central event management (single source of truth)
- `src/core/EventStore.ts` -- In-memory event database with indexes
- `src/calendars/FullNoteCalendar.ts` -- Full note calendar implementation
- `src/calendars/DailyNoteCalendar.ts` -- Daily note calendar implementation
- `src/ui/view.ts` -- Main calendar view integration with FullCalendar.js
- `src/types/schema.ts` -- Zod schemas for data validation

### Build System Details
- **Bundler**: esbuild with custom configuration
- **CSS Handling**: Automatically renames main.css to styles.css for Obsidian compatibility
- **TypeScript**: Strict type checking with `tsc --noEmit`
- **External Dependencies**: FullCalendar.js, React, Luxon, and others bundled but Obsidian APIs marked as external

### Testing Framework

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [obsidian-full-calendar-remastered/plugin-full-calendar](https://github.com/obsidian-full-calendar-remastered/plugin-full-calendar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
