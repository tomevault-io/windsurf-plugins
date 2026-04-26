---
trigger: always_on
description: Fork of [OpenCode](https://github.com/crisogray/claudius) focused on desktop app development.
---

# ADE: Agent Development Environment

Fork of [OpenCode](https://github.com/crisogray/claudius) focused on desktop app development.

## Architecture

### Package Structure

```
packages/
  desktop/     # Tauri v2 shell, sidecar management
  app/         # SolidJS frontend, context providers, pages
  opencode/    # Agent server, tools, snapshots (runs as sidecar)
  ui/          # Shared components, themes, @pierre/diffs integration
  sdk/         # Auto-generated API client, SSE events
  util/        # Helpers (encoding, binary search, retry)
```

### Desktop App (`packages/desktop`)

Tauri v2 app that wraps the frontend and manages the opencode CLI as a sidecar:

- `src-tauri/src/lib.rs` - Main app logic, sidecar spawning, window creation
- `src-tauri/src/cli.rs` - CLI installation/sync to `~/.opencode/bin/opencode`
- `src/index.tsx` - Entry point, renders app inside `ServerGate`

**Sidecar flow:**

```
Tauri app starts
  -> spawn_sidecar("opencode serve --port={port}")
  -> inject port via window.__OPENCODE__
  -> frontend connects via HTTP/SSE to localhost:{port}
```

**Tauri commands:** `kill_sidecar`, `install_cli`, `ensure_server_started`

### App Package (`packages/app`)

SolidJS frontend with nested context providers:

```
PlatformProvider (web vs desktop abstraction)
  ServerProvider (connection management)
    GlobalSDKProvider (SDK client + global events)
      GlobalSyncProvider (projects, providers state)
        Router
          Layout (sidebar, navigation)
            DirectoryLayout (per-directory scope)
              SDKProvider, SyncProvider (directory-scoped)
                LocalProvider (model, agent, files)
                  TerminalProvider, FileProvider, PromptProvider
                    Session page
```

**Key files:**

- `src/pages/layout.tsx` - Sidebar with projects/sessions, drag-drop reordering
- `src/pages/session.tsx` - Main work surface: messages, prompt input, review tabs
- `src/context/sync.tsx` - State management, SSE event handling

**Current layout:**

```
+---------------+---------------------------------------------+
|   SIDEBAR     |              MAIN AREA                      |
|   resizable   |                                             |
+---------------+---------------------------------------------+
| Projects      | Session header                              |
|   Sessions    | Message list (SessionTurn components)       |
|               | Prompt input (bottom)                       |
|               |                                             |
|               | [Optional right panel: Review/Context/File] |
+---------------+---------------------------------------------+
```

### UI Package (`packages/ui`)

42 shared components built on Kobalte primitives:

- **Core:** button, icon-button, checkbox, switch, text-field, select, dialog, popover, tooltip, dropdown-menu, tabs, accordion
- **Domain:** diff, code, markdown, session-turn, session-review, message-part
- **Patterns:** `data-component`, `data-variant`, `data-slot` attributes for CSS targeting

**Diff rendering (`src/pierre/`):**

- Uses `@pierre/diffs` for syntax-highlighted diffs
- `FileDiff` class with worker pool for highlighting
- Supports `DiffLineAnnotation<T>` for inline annotations (currently unused)
- SSR support via `preloadMultiFileDiff` and hydration

**Theme system:**

- 12 built-in themes (dracula, nord, catppuccin, etc.)
- OKLCH-based colors with CSS custom properties
- Runtime switching via `ThemeProvider`

### SDK Package (`packages/sdk`)

Auto-generated from OpenAPI spec:

**Client methods:**

- `session.*` - CRUD, prompt, abort, revert, diff, messages
- `provider.*` - List, auth, OAuth
- `file.*` - List, read, status
- `find.*` - Text, files, symbols search
- `mcp/lsp.*` - Server status

**SSE events (36+ types):**

- `session.created/updated/deleted/status/error`
- `message.updated/removed`, `message.part.updated/removed`
- `permission.asked/replied`
- `file.edited`, `pty.*`, `lsp.updated`, `vcs.branch.updated`

### OpenCode Package (`packages/opencode`)

Hono server with agent system:

**Server (`src/server/`):**

- REST API with OpenAPI docs
- SSE at `/global/event` and `/event` (directory-scoped)
- WebSocket for PTY terminals

**Agents (`src/agent/`):**

- Built-in: `general`, `explore`, `plan`, `build`, `compaction`, `title`, `summary`
- Permission system with allow/deny/ask rules
- Custom agents via config

**Tools (`src/tool/`):**

- Core: `bash`, `read`, `write`, `edit`, `glob`, `grep`, `task`, `webfetch`
- Edit uses fuzzy matching chain for robust replacements
- Custom tools from `tool/*.ts` in config directories
- MCP tools integrated via `MCP.tools()`

**Snapshots (`src/snapshot/`):**

- Shadow git repo at `~/.opencode/data/snapshot/{projectID}`
- `track()` creates tree hash of worktree state
- `patch(hash)` returns changed files since snapshot
- `diffFull(from, to)` returns full diff with before/after content

**Diff schema:**

```typescript
interface FileDiff {
  file: string
  before: string // Full content before
  after: string // Full content after
  additions: number
  deletions: number
}
```

## Key Patterns

### State Management

- Two-tier: Global (projects, providers) + Directory-scoped (sessions, messages, parts)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crisogray/claudius](https://github.com/crisogray/claudius) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
