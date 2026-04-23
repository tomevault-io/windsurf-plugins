---
trigger: always_on
description: Open-source desktop client for the Agent Client Protocol. Uses the `@anthropic-ai/claude-agent-sdk` to programmatically manage Claude sessions via `query()`. Supports multiple concurrent sessions with persistent chat history, project workspaces, background agents, tool permissions, and context compaction.
---

# Harnss

Open-source desktop client for the Agent Client Protocol. Uses the `@anthropic-ai/claude-agent-sdk` to programmatically manage Claude sessions via `query()`. Supports multiple concurrent sessions with persistent chat history, project workspaces, background agents, tool permissions, and context compaction.

## Tech Stack

- **Runtime**: Electron 40 (main process) + React 19 (renderer)
- **Build**: Vite 7, TypeScript 5.9, tsup (electron TS→JS)
- **Styling**: Tailwind CSS v4 + ShadCN UI (includes Preflight — no CSS resets needed)
- **UI Components**: ShadCN (Button, Badge, ScrollArea, Tooltip, Collapsible, Separator, DropdownMenu, Avatar)
- **Icons**: lucide-react
- **Markdown**: react-markdown + remark-gfm + react-syntax-highlighter + @tailwindcss/typography
- **Diff**: diff (word-level diff rendering)
- **Glass effect**: electron-liquid-glass (macOS Tahoe+ transparency)
- **SDK**: @anthropic-ai/claude-agent-sdk (ESM-only, async-imported from CommonJS)
- **Terminal**: node-pty (main process) + @xterm/xterm + @xterm/addon-fit (renderer)
- **Browser**: Electron `<webview>` tag (requires `webviewTag: true` in webPreferences)
- **Package manager**: pnpm
- **Path aliases**: `@/` → `./src/`, `@shared/` → `./shared/`

## Project Structure

```
shared/
└── types/             # Types shared between electron and renderer processes
    ├── codex-protocol/  # Auto-generated Codex protocol types (from codex app-server)
    │   ├── v2/          # Modern v2 API types
    │   └── serde_json/  # JSON value types
    ├── codex.ts         # Codex type re-exports with Codex-prefixed aliases
    ├── engine.ts        # EngineId, AppPermissionBehavior, SlashCommand, RespondPermissionFn
    ├── acp.ts           # ACP session update types
    └── registry.ts      # Agent registry types

electron/
├── dist/       # tsup build output (gitignored)
└── src/
    ├── ipc/    # IPC handlers (claude-sessions, projects, sessions, settings, terminal, git, etc.)
    └── lib/    # Main-process utilities (logger, async-channel, data-dir, app-settings, sdk, error-utils, etc.)

src/
├── components/
│   ├── git/           # GitPanel decomposed (GitPanel, RepoSection, BranchPicker, CommitInput, etc.)
│   ├── mcp-renderers/ # MCP tool renderers (jira, confluence, atlassian, context7)
│   ├── tool-renderers/# Built-in tool renderers (BashContent, EditContent, TaskTool, etc.)
│   ├── sidebar/       # AppSidebar decomposed (ProjectSection, SessionItem, CCSessionList)
│   ├── lib/           # Component-local utilities (tool-metadata, tool-formatting)
│   ├── settings/      # Settings sub-views + shared SettingRow/SettingsSelect
│   └── ui/            # ShadCN base components (auto-generated)
├── hooks/
│   ├── session/       # useSessionManager decomposed (lifecycle, persistence, draft, revival, queue)
│   └── ...            # React hooks (useEngineBase, useClaude, useAppOrchestrator, usePanelResize, etc.)
├── lib/               # Renderer utilities (protocol, streaming-buffer, message-factory, background stores, etc.)
└── types/             # Renderer-side types (protocol, ui, window.d.ts) + re-export shims for shared/
```

## How to Run

```bash
pnpm install
pnpm dev       # Starts Vite dev server + tsup watch + Electron
pnpm build     # tsup (electron/) + Vite (renderer) production build
pnpm start     # Run Electron with pre-built dist/
```

**Dev logs**: Main process logs go to `logs/main-{timestamp}.log` (dev) or `{userData}/logs/main-{timestamp}.log` (packaged). Check the latest file with `ls -t logs/main-*.log | head -1 | xargs cat`.

## Architecture

### SDK-Based Session Management

The main process uses `@anthropic-ai/claude-agent-sdk` (ESM-only, loaded via `await import()`). Each session runs a long-lived SDK `query()` with an `AsyncChannel` for multi-turn input.

**Session Map**: `Map<sessionId, { channel, queryHandle, eventCounter, pendingPermissions }>`

- `channel` — AsyncChannel (push-based async iterable) for sending user messages to SDK
- `queryHandle` — SDK query handle for interrupt/close/setPermissionMode
- `pendingPermissions` — Map<requestId, { resolve }> for bridging SDK permission callbacks to UI

**IPC API — Claude Sessions:**

- `claude:start(options)` → spawns SDK query with AsyncChannel, returns `{ sessionId, pid }`
  - Options: `cwd`, `model`, `permissionMode`, `resume` (session continuation)
  - Configures `canUseTool` callback for permission bridging
  - Thinking: `{ type: "enabled", budgetTokens: 16000 }`
- `claude:send({ sessionId, message })` → pushes user message to session's AsyncChannel
- `claude:stop(sessionId)` → closes channel + query handle, removes from Map
- `claude:interrupt(sessionId)` → denies all pending permissions, calls `queryHandle.interrupt()`
- `claude:permission_response(sessionId, requestId, ...)` → resolves pending permission Promise
- `claude:set-permission-mode(sessionId, mode)` → calls `queryHandle.setPermissionMode()`
- `claude:generate-title(message, cwd?)` → one-shot Haiku query for chat title

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenSource03/harnss](https://github.com/OpenSource03/harnss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
