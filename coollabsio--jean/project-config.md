---
trigger: always_on
description: This repository is a template with sensible defaults for building Tauri React apps.
---

# AI Agents

## Current Status

@CLAUDE.local.md

## Overview

This repository is a template with sensible defaults for building Tauri React apps.

## Core Rules

### Codex App Server schema

If you need to check some codex app-server related things, use "codex app-server generate-json-schema --out ./codex-schema" to generate schema and check local dir ./codex-schema for schemas.

### New Sessions

- Read @docs/tasks.md for task management
- Review `docs/developer/architecture-guide.md` for high-level patterns
- Check `docs/developer/` for system-specific patterns (command-system.md, performance-patterns.md, etc.)
- Check git status and project structure

### Development Practices

**CRITICAL:** Follow these strictly:

1. **Read Before Editing**: Always read files first to understand context
2. **Follow Established Patterns**: Use patterns from this file and `docs/developer`
3. **Senior Architect Mindset**: Consider performance, maintainability, testability
4. **Batch Operations**: Use multiple tool calls in single responses
5. **Match Code Style**: Follow existing formatting and patterns
6. **Test Coverage**: Write comprehensive tests for business logic
7. **Quality Gates**: Run `bun run check:all` after significant changes
8. **Dev Server**: You may start the dev server (`bun run tauri dev`) when needed
9. **No Unsolicited Commits**: Only when explicitly requested
10. **Documentation**: Update relevant `docs/developer/` files for new patterns
11. **Removing files**: Always use `rm -f`

**CRITICAL:** Use Tauri v2 docs only. Always use modern Rust formatting: `format!("{variable}")`

## Architecture Patterns (CRITICAL)

### State Management Onion

```
useState (component) → Zustand (global UI) → TanStack Query (persistent data)
```

**Decision**: Is data needed across components? → Does it persist between sessions?

### Performance Pattern (CRITICAL)

```typescript
// ✅ GOOD: Use getState() to avoid render cascades
const handleAction = useCallback(() => {
  const { data, setData } = useStore.getState()
  setData(newData)
}, []) // Empty deps = stable

// ❌ BAD: Store subscriptions cause cascades
const { data, setData } = useStore()
const handleAction = useCallback(() => {
  setData(newData)
}, [data, setData]) // Re-creates constantly
```

### Event-Driven Bridge

- **Rust → React**: `app.emit("event-name", data)` → `listen("event-name", handler)`
- **React → Rust**: `invoke("command_name", args)` with TanStack Query
- **Commands**: All actions flow through centralized command system

### Documentation & Versions

- **Context7 First**: Always use Context7 for framework docs before WebSearch
- **Version Requirements**: Tauri v2.x, shadcn/ui v4.x, Tailwind v4.x, React 19.x, Zustand v5.x, Vite v7.x, Vitest v4.x

### Important Findings & Learnings

**Document discoveries here.** When encountering major/minor findings during development, ask the user if they should be saved to this file for future reference.

#### Rust-TypeScript Serialization Convention

**CRITICAL:** There are two patterns for Rust-TypeScript serialization. Pick ONE per struct and be consistent.

**Pattern A: snake_case (for persisted/settings data)**

- Used for: `AppPreferences`, `UIState`, and other persisted data
- Rust structs use snake_case by default (e.g., `active_worktree_id`)
- TypeScript interfaces must match exactly (e.g., `active_worktree_id`, NOT `activeWorktreeId`)
- See `src/types/preferences.ts` and `src/types/ui-state.ts` for examples

**Pattern B: camelCase with `#[serde(rename_all = "camelCase")]` (for API/command data)**

- Used for: Data passed between frontend and Tauri commands (e.g., `IssueContext`, `PullRequestContext`)
- Add `#[serde(rename_all = "camelCase")]` to Rust struct
- TypeScript uses standard camelCase (e.g., `headRefName`, `baseRefName`)
- See `src-tauri/src/projects/github_issues.rs` for examples

**Common error:** `invalid args for command: missing field 'field_name'`

- This means Rust expects snake_case but frontend sent camelCase (or vice versa)
- Fix: Add `#[serde(rename_all = "camelCase")]` to the Rust struct, OR change TypeScript to snake_case

#### UI State Persistence Pattern

Session-specific UI state (e.g., answered questions, fixed review findings) must be persisted via the existing Tauri backend system, not Zustand middleware:

1. **Add fields to** `src/types/ui-state.ts` (TypeScript interface, use `snake_case`)
2. **Add fields to** `src-tauri/src/lib.rs` (Rust `UIState` struct with `#[serde(default)]`)
3. **Update** `src/hooks/useUIStatePersistence.ts`:
   - Extract state in `getCurrentUIState()` (map camelCase store → snake_case UIState, convert Sets to arrays)
   - Restore state in initialization effect (map snake_case UIState → camelCase store, convert arrays back to Sets)
   - Track changes in subscription effect to trigger saves

**Key insight**: The `hasFollowUpMessage` check in `ChatWindow.tsx` (checks if a user message follows an assistant message) is meant as a fallback but may have timing issues with TanStack Query. Persisting state directly provides reliable rendering.

#### Zustand Getter Function Anti-Pattern

**CRITICAL:** Never subscribe to a getter function and call it directly in JSX. This creates NO subscription to the underlying data.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coollabsio/jean](https://github.com/coollabsio/jean) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
