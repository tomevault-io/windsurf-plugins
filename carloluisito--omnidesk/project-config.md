---
trigger: always_on
description: Multi-provider Electron desktop application for AI coding CLIs, with multi-session terminals, split views, and agent team visualization. Supports Claude Code, Codex CLI, and future providers via a pluggable provider abstraction layer.
---

# OmniDesk

Multi-provider Electron desktop application for AI coding CLIs, with multi-session terminals, split views, and agent team visualization. Supports Claude Code, Codex CLI, and future providers via a pluggable provider abstraction layer.

## Workflow Rule

**Always run the `requirements-clarifier` agent (via the Task tool) on the user's initial prompt before planning or implementing.** This ensures requirements are analyzed, ambiguities are surfaced, and acceptance criteria are established before any work begins. Skip only for trivial tasks (typo fixes, single-line changes, or purely informational questions).

## Tech Stack

Electron 28 | React 18 | TypeScript | xterm.js | node-pty | Tailwind CSS | reactflow

## Architecture

```
┌─────────────────────────────────────────────┐
│  Main Process (Node.js)                     │
│  15 managers + IPC handlers + session pool  │
└──────────────────┬──────────────────────────┘
                   │ IPC (~191 methods)
┌──────────────────┴──────────────────────────┐
│  Preload (auto-derived context bridge)      │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────┴──────────────────────────┐
│  Renderer (React 18)                        │
│  Hooks → Components → UI                   │
└─────────────────────────────────────────────┘
```

**3-layer pattern per domain:** Manager (main) → Hook (renderer) → Components (renderer)

**IPC contract** (`src/shared/ipc-contract.ts`) is the single source of truth — ~191 methods. The preload bridge and `ElectronAPI` type are auto-derived from it.

**Provider abstraction**: `IProvider` interface (`src/main/providers/`) decouples CLI specifics from session management. `CLIManager` delegates to the active provider for command building, environment variables, and model detection. Default provider is Claude.

## Domain Map

| Domain | Main (manager) | Renderer (hook + UI) | Shared types | IPC prefix |
|--------|---------------|---------------------|-------------|------------|
| Sessions | session-manager, cli-manager, session-pool, session-persistence | useSessionManager, Terminal | ipc-types.ts | `session:*` |
| Split View | settings-persistence | useSplitView, SplitLayout, PaneHeader, PaneSessionPicker | ipc-types.ts | `settings:*` |
| Agent Teams | agent-team-manager | useAgentTeams, useAutoTeamLayout, useMessageStream, TeamPanel, TaskBoard, MessageStream, AgentGraph | ipc-types.ts, message-parser.ts | `teams:*` |
| Templates | prompt-templates-manager, built-in-actions | useCommandPalette, CommandPalette, TemplateEditor | types/prompt-templates.ts | `template:*` |
| Custom Commands | custom-command-manager | useCustomCommands, CustomCommandDialog, CustomCommandPanel, CustomCommandParameterDialog | types/custom-command-types.ts | `customCommands:*` |
| History | history-manager | useHistory, HistoryPanel | types/history-types.ts | `history:*` |
| Checkpoints | checkpoint-manager, checkpoint-persistence | useCheckpoints, CheckpointPanel, CheckpointDialog | types/checkpoint-types.ts | `checkpoint:*` |
| Quota | quota-service | useQuota, BudgetPanel, BudgetSettings | ipc-types.ts | `quota:*`, `burnRate:*` |
| Drag-Drop | file-dragdrop-handler, file-utils | useDragDrop, DragDropOverlay, DragDropContextMenu, DragDropSettings | ipc-types.ts | `dragdrop:*` |
| Workspaces | settings-persistence | SettingsDialog | ipc-types.ts | `workspace:*` |
| Atlas | atlas-manager | useAtlas, AtlasPanel | types/atlas-types.ts | `atlas:*` |
| Git | git-manager | useGit, useDiffViewer, GitPanel, DiffViewer, DiffFileNav, DiffViewerHeader, DiffContentArea, CommitDialog, WorktreePanel, WorktreeCleanupDialog, diff-parser | types/git-types.ts | `git:*` |
| Playbooks | playbook-manager, playbook-executor, built-in-playbooks | usePlaybooks, PlaybookPicker, PlaybookParameterDialog, PlaybookProgressPanel, PlaybookPanel, PlaybookEditor | types/playbook-types.ts | `playbook:*` |
| Tunnels | tunnel-manager | useTunnel, TunnelPanel, TunnelCreateDialog, TunnelRequestLogs | types/tunnel-types.ts | `tunnel:*` |
| Providers | provider-registry, claude-provider, codex-provider | useProvider | types/provider-types.ts | `provider:*` |
| Session Sharing | sharing-manager | useSessionSharing, ShareSessionDialog, JoinSessionDialog, ObserverToolbar, ObserverMetadataSidebar, ShareManagementPanel, ShareIndicator, ControlRequestDialog | types/sharing-types.ts | `sharing:*` |
| Window | index.ts | ConfirmDialog, SettingsDialog, AboutDialog, TitleBarBranding | ipc-types.ts | `window:*`, `dialog:*` |

## Adding a New IPC Method

1. Add entry to `src/shared/ipc-contract.ts` (in `IPCContractMap`)
2. Add handler in `src/main/ipc-handlers.ts` using `registry.handle()` / `registry.on()`

That's it. The preload bridge and types auto-derive.

## Adding a New Domain

1. Create `src/main/<domain>-manager.ts` (or `src/main/<domain>/` directory for multi-file domains like Providers)
2. Create `src/renderer/hooks/use<Domain>.ts`
3. Create component(s) in `src/renderer/components/`
4. Add IPC methods to `ipc-contract.ts` with `<domain>:*` prefix

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carloluisito/omnidesk](https://github.com/carloluisito/omnidesk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
