---
trigger: always_on
description: Desktop app for browsing AI coding sessions. Tauri 2.0 + Solid.js + Rust + SQLite FTS5.
---

# CC Session

Desktop app for browsing AI coding sessions. Tauri 2.0 + Solid.js + Rust + SQLite FTS5.

## Commands

```bash
npm run tauri dev             # Dev with hot reload
npm run tauri build           # Production build
cd src-tauri && cargo clippy  # Rust lint
cd src-tauri && cargo test    # Rust tests
npx tsc --noEmit              # TS type check
npm run lint                  # ESLint
npm run format:check          # Prettier check
./scripts/release.sh <version> # Bump, commit, tag, push → triggers CI release
```

## Project Layout

```
src/                       # Solid.js frontend (components, stores, i18n, lib, styles)
src-tauri/src/
  providers/               # claude/, codex/, antigravity/, kimi/, opencode/, cursor/, cc_mirror.rs
  commands/                # sessions.rs, settings.rs, trash.rs, terminal.rs, usage.rs, search.rs, file_access.rs
  services/                # provider_snapshots, session_lifecycle, session_resolution, source_sync, image_cache, terminal (platform launchers), caches
  exporter/                # json.rs, markdown.rs, html.rs, templates.rs
  db/                      # mod.rs, sync.rs, row_mapper.rs, queries.rs + queries/{sessions,tree,favorites,search,usage}.rs
  provider.rs + provider/  # SessionProvider machinery: traits.rs, plan.rs, state.rs, tokens.rs, catalog.rs, trash.rs
  tool_metadata.rs + tool_metadata/  # names.rs, summary.rs, result.rs, build.rs
  indexer.rs  watcher.rs  models.rs  provider_utils.rs  trash_state.rs  pricing.rs
src/components/           # feature dirs (Editor/, SessionView/, Explorer/, UsagePanel/, MessageBubble/, TrashView/, Settings/) + small flat components, icons.tsx
src/stores/               # editorGroups, settings, search, selection, providerSnapshots, updater, favorites, toast, theme, usageView
src/lib/                  # tauri.ts (IPC), tools/, formatters, tree-builders, heatmap, diff, subagent, image-cache, provider-watch
src/styles/               # variables.css (theme tokens) + per-feature files (layout, editor, explorer, session, tools, code, messages, modals, search, trash, settings, feedback, images, utilities, usage); cascade order lives in index.css
```

## Editor UI Architecture

```
App
├── ActivityBar                        # Left icon bar (explorer, favorites, usage, trash, etc.)
├── [Left panel — conditional on activeView()]
│   ├── Explorer                       # Session tree with single-click=preview, double-click=pin
│   ├── FavoritesView / TrashView / BlockedView / UsagePanel / SettingsPanel
├── SearchPanel                        # In titlebar-right, not in left panel
├── EditorGroupsContainer             # Manages split view layout (max 4 groups)
│   ├── SplitHandle                    # Draggable resize between groups
│   └── EditorArea (per group)
│       ├── TabBar                     # Tabs with preview italic, overflow chevron dropdown
│       └── SessionView (per tab)      # Full session viewer (messages, toolbar, search)
└── StatusBar                          # Index count, today's cost, provider info
```

### Editor Groups Store (`src/stores/editorGroups.ts`)

Central store for tabs, split view, and preview mode. All state is immutable (spread updates).

```typescript
interface EditorGroup {
  id: string;
  tabs: SessionRef[];          // open sessions in this group
  activeTabId: string | null;  // currently visible tab
  previewTabId: string | null; // at most one preview (unpinned) tab
  flexBasis: number;           // width percentage for split view
}
```

Key functions:
- `openSession(session)` — pin-open a tab (or focus if already open, auto-pins preview)
- `openPreview(session)` — open as preview tab (italic, replaces previous preview in group)
- `pinTab(sessionId)` — promote preview to permanent
- `splitToRight(sessionId)` — move tab to right group (creates new group if needed)
- `moveTabToGroup(sessionId, targetGroupId, insertIndex?)` — drag-drop between groups
- `createGroupFromDrop(sessionId)` — drop zone creates new rightmost group

### Preview Mode (VSCode-style)

- **Explorer single click** → `openPreview()` — italic tab, replaced by next preview
- **Explorer double click** → `openSession()` — permanent pinned tab
- **Double-click on preview tab** → `pinTab()` — pins it
- **Search/Favorites/Subagent open** → `openSession()` — always pinned
- Each group has at most one `previewTabId`; replacement removes old preview tab from array
- SessionView is wrapped in `<Show when={session().id} keyed>` to force remount on preview replacement (prevents stale local state: filters, search, watch, favorite)

### Tab Overflow

- Hidden-scrollbar scroll container with mouse wheel horizontal scroll
- `ResizeObserver` + `props.tabs` array reference change triggers overflow detection
- `»` chevron button with dropdown listing all tabs (active indicator, italic for preview)

## Event System

Custom DOM events for cross-component communication:

| Event | Dispatcher | Handler | Purpose |
|-------|-----------|---------|---------|
| `open-subagent` | ToolMessage "↗ Open" button | App.handleOpenSubagent | Navigate to child session by agentId/nickname/description |
| `usage-data-changed` | SessionView (on favorite/export) | App (refreshes usage panel) | Sync usage stats |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tyql688/cc-session](https://github.com/tyql688/cc-session) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
