---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This file is the primary guidance document for the ltk-manager codebase.

## Commands

All commands run from the repo root.

```bash
# Full dev mode (Rust backend + React frontend with hot reload)
pnpm tauri dev

# Frontend only (skip Rust rebuild, faster iteration on UI)
pnpm dev

# Type check / lint / format / all three
pnpm typecheck
pnpm lint
pnpm format
pnpm check          # typecheck + lint + format:check

# Production build
pnpm tauri build

# Rust-only operations (from workspace root)
cargo clippy -p ltk-manager
cargo fmt -p ltk-manager

# Verbose backend logging
RUST_LOG=ltk_manager=trace,tauri=info pnpm tauri dev
```

## Editing Rules

**Always read files before editing them.** Never assume file contents from memory or prior context. When making bulk edits across multiple files, read all target files first, then perform edits.

## Code Style

From `.cursorrules`: avoid trivially descriptive comments. Only comment non-obvious business logic, workarounds, edge cases, or "why" decisions. Document all public Rust APIs with `///` doc comments.

**No redundant comments.** Do not add inline comments that restate what the code already expresses. If the code is descriptive enough (clear variable names, well-known patterns like temp-file-then-rename, obvious API calls), leave it uncommented. This applies to AI-generated code and suggestions too — strip narration comments before committing.

### JSX Conditional Rendering

**Avoid ternary operators in JSX.** Use early returns or `{condition && <Component />}` instead.

```tsx
// Good — early return
if (isLoading) return <LoadingState />;
if (error) return <ErrorState error={error} />;
return <Content />;

// Good — single-line conditional
{
  hasItems && <ItemList items={items} />;
}

// Bad — ternary in JSX
{
  isLoading ? <LoadingState /> : error ? <ErrorState /> : <Content />;
}
```

### Import Conventions

**Always import from barrel exports, never from subdirectories.** This keeps import paths stable and encapsulates internal structure.

- **Global components:** import from `@/components`, not `@/components/Button`, `@/components/Toast`, etc.
- **Modules:** import from `@/modules/{module}`, not `@/modules/{module}/components` or `@/modules/{module}/api`.

```ts
// Good
import { Button, IconButton, useToast } from "@/components";
import { ModCard, useInstalledMods } from "@/modules/library";

// Bad — reaches into internals
import { Button } from "@/components/Button";
import { useToast } from "@/components/Toast";
import { ModCard } from "@/modules/library/components";
```

### State Consumption — Hooks Over Prop Drilling

**Consume global state (hooks, queries, stores) directly in the component that needs it.** Do not drill Zustand state, TanStack Query data, or mutation callbacks through intermediate components as props.

- Patcher status → call `usePatcherStatus()` in the component that checks it
- Mod toggle/uninstall → call `useToggleMod()` / `useUninstallMod()` in `ModCard`, not passed from a parent
- Folder toggle → call `useFolderToggle()` in `FolderRow`/`FolderCard`, not received as a prop

TanStack Query deduplicates identical queries, so multiple components calling the same hook is efficient and correct.

**Exception:** Props are appropriate for coordinating parent-owned UI state (e.g., `onViewDetails` that opens a sibling dialog, `onReorder` where reorder target varies by context).

## Backend (Rust) — `src-tauri/src/`

### Module Layout

- `main.rs` — Tauri setup, command registration in `generate_handler![]`, logging init
- `error.rs` — `AppError`, `AppErrorResponse`, `IpcResult<T>`, `MutexResultExt`
- `state.rs` — `SettingsState(Mutex<Settings>)`, settings persistence
- `commands/` — `#[tauri::command]` wrappers (one file per domain: `mods.rs`, `profiles.rs`, `patcher.rs`, `settings.rs`, `workshop.rs`, `shell.rs`, `app.rs`)
- `mods/mod.rs` — Business logic for mod install/uninstall/toggle, profile CRUD, library index management
- `overlay/` — Overlay building, content providers (`modpkg_content.rs`, `fantome_content.rs`)
- `patcher/` — Patcher lifecycle (start/stop/status), thread management with `Arc<AtomicBool>` stop flag
- `legacy_patcher/` — FFI integration with `cslol-dll.dll`

### State

Two Tauri-managed states:

- `SettingsState` — App settings (league path, storage path, theme). Access via `State<SettingsState>`, lock with `.0.lock().mutex_err()?.clone()`.
- `PatcherState` — Patcher thread handle and stop flag. Access via `State<PatcherState>`.

### Error Codes

`ErrorCode` enum variants (serialized as `SCREAMING_SNAKE_CASE`): `Io`, `Serialization`, `Modpkg`, `Fantome`, `LeagueNotFound`, `InvalidPath`, `ModNotFound`, `ValidationFailed`, `InternalState`, `MutexLockFailed`, `PatcherRunning`, `Unknown`, `WorkshopNotConfigured`, `ProjectNotFound`, `ProjectAlreadyExists`, `PackFailed`, `Wad`, `Zip`.

Errors can carry JSON context: `AppErrorResponse::new(code, msg).with_context(json!({ "modId": id }))`.

## Frontend (React + TypeScript) — `src/`

### Key Files


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeagueToolkit/ltk-manager](https://github.com/LeagueToolkit/ltk-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
