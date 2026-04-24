---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GSD Console - Terminal UI for viewing GSD (Get Shit Done) project status. Displays roadmap progress, phase details, and todos in a keyboard-navigable interface built with Ink (React for terminals).

## Commands

```bash
# Development
bun run dev           # Run with hot reload
bun start             # Run once
bun start --only roadmap  # Single view mode

# Validation
bun run typecheck     # TypeScript check (tsc --noEmit)
bun run lint          # Biome linting
bun run lint:fix      # Auto-fix lint issues

# Testing
bun test              # Run all tests
bun test test/lib/parser.test.ts  # Run single test file
bun run test:coverage # Run with coverage
```

## Architecture

### Data Flow

```
.planning/ files → parser.ts → useGsdData hook → App → TabLayout → Views
                                    ↑
                        useFileWatcher (auto-refresh)
```

- `lib/parser.ts` - Parses ROADMAP.md, STATE.md, and phase directories
- `hooks/useGsdData.ts` - Loads and caches parsed data
- `hooks/useFileWatcher.ts` - Watches .planning/ for changes, triggers refresh
- `hooks/useChangeHighlight.ts` - Tracks recently changed items for visual highlighting

### Component Hierarchy

```
App.tsx
├── Header (project name, progress)
├── TabLayout (tab switching: 1/2/3/4 or Tab)
│   ├── RoadmapView (phase list with expand/collapse)
│   ├── PhaseView (single phase detail)
│   ├── TodosView (todo list)
│   └── BackgroundView (job queue status)
├── Footer (context-sensitive keybindings)
├── CommandPalette (: key, fuzzy search with Tab completion)
├── ExecutionModePrompt (headless/interactive/primary selection)
├── SessionPicker (c key, connect to OpenCode sessions)
├── FilePicker (e key, multi-file selection for editor)
└── ToastContainer (notifications)
```

### Navigation Hooks

- `useVimNav` - Vim-style navigation (j/k, gg/G, Ctrl+d/u) for any list
- `useTabNav` - Tab key and number key (1/2/3) switching between views
- `useTabState` - Ref-based per-tab state persistence (no re-renders on save)

### Key Patterns

**Input handling:** Use Ink's `useInput` hook with `isActive` flag to prevent overlapping handlers. Only one component should handle input at a time.

**State lifting:** Selection state (phase number, todo ID) is lifted to App.tsx for editor integration. Tab-specific state (expanded phases, scroll) uses `useTabState` ref storage.

**Overlays:** Command palette and file picker render with `position="absolute"` and disable underlying input handlers via `isActive` prop drilling.

## Code Style

- **Formatting:** Biome with tabs, single quotes, semicolons, 100 char line width
- **Commits:** Conventional commits with lowercase subject (`feat(scope): add feature`)
- **Unused vars:** Prefix with underscore (`_unusedParam`) or use `void expression`
- **Git hooks:** Lefthook runs biome check and typecheck on pre-commit, commitlint on commit-msg

## Testing Approach

**Philosophy:** Use real data instead of mocks where possible, test behavior rather than hardcoded values.

### Test Data Strategy

- **`.planning/` directory is read-only test data** - Use actual planning files instead of memfs mocking
  - Parser tests use memfs for isolated filesystem testing
  - Hook tests read from real `.planning/` directory to test integration
  - Tests won't break when adding phases/plans to the project

### Mocking Guidelines

- **Mock only what's necessary** - Most tests use real data, mocks only for error cases
  - Don't mock parser functions in hook tests - let them read from `.planning/`
  - Mock `node:fs` only for specific error scenarios (missing directory)
  - Avoid mock conflicts between test files (e.g., parser tests vs hook tests)

### Flexible Assertions

- **Test behavior, not exact values** - Use `toBeGreaterThan(0)` instead of `toBe(5)`
  - Tests remain valid when adding more phases: `expect(phases.length).toBeGreaterThan(0)`
  - Tests remain valid when adding more plans: `expect(state.progress).toBeGreaterThanOrEqual(0)`
  - Verify structure exists: `expect(phase).toHaveProperty('number')`

### Example: Resilient Hook Test

```tsx
// ✅ Good: Flexible to project growth
test('loads and parses planning documents successfully', async () => {
  const data = useGsdData('.planning');
  
  // These assertions won't break when adding phases
  expect(data.phases.length).toBeGreaterThan(0);
  expect(data.phases[0]).toHaveProperty('number');
  expect(data.phases[0]).toHaveProperty('name');
});

// ❌ Bad: Brittle to project growth
test('loads 5 phases', async () => {
  const data = useGsdData('.planning');
  
  // This breaks when you add a 6th phase
  expect(data.phases).toBe(5);
});
```

### Example: Parser Test with memfs

```tsx
// Parser tests use memfs for isolated testing
import { fs } from 'memfs';
vi.mock('node:fs', () => fs);

test('parses phase from markdown', () => {
  vol.fromJSON({
    '.planning/ROADMAP.md': '### Phase 1: Test\n**Goal**: Build',
  });
  
  const phases = parseRoadmap(content, '.planning/phases');
  expect(phases).toHaveLength(1);
});
```

## Avoiding Flicker


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Codesushi-com) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
