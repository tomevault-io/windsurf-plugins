---
trigger: always_on
description: Electron desktop app (React 19 + TypeScript + Zustand) for managing Git worktrees and multi-agent AI sessions (Claude, Codex, Cursor, Copilot, Gemini, Grok, Hermes, Droid). Three-panel layout: sidebar (projects/worktrees), center (chat + file editor + embedded terminals), right (files/diffs/checks/terminal/notes/simulator/search). Includes a Mission Control kanban board for cross-worktree oversight and a mobile companion server for physical device pairing.
---

# Braid

Electron desktop app (React 19 + TypeScript + Zustand) for managing Git worktrees and multi-agent AI sessions (Claude, Codex, Cursor, Copilot, Gemini, Grok, Hermes, Droid). Three-panel layout: sidebar (projects/worktrees), center (chat + file editor + embedded terminals), right (files/diffs/checks/terminal/notes/simulator/search). Includes a Mission Control kanban board for cross-worktree oversight and a mobile companion server for physical device pairing.

## Architecture

- **Main process** (`src/main/`): Electron window, IPC handlers, services
- **Preload** (`src/preload/index.ts`): Context-isolated bridge exposing `api.{storage,git,agent,pty,simulator,scripts,templates,windowCapture,github,jira,sessions,shell,files,search,clipboard,claudeCli,dialog,window,claudeConfig,menu,dock,notes,lsp,updater,drag,settings,mobile}`
- **Renderer** (`src/renderer/`): React UI with Zustand stores
- **Shared** (`src/shared/`): Types/protocols shared between main and renderer (search, templates, mobile-protocol, projectName)
- **IPC flow**: changes must be threaded through all 3 layers (`src/main/ipc.ts` -> `src/preload/index.ts` -> `src/renderer/lib/ipc.ts`)

## Key Directories

```
src/main/services/
  git/              # Modular: core, branches, config, status, operations, worktrees, snapshots, types
  agentWorker/      # Worker thread management: core, errorClassifier, mcp, tools
  agentHooks/       # Multi-agent hook system: per-agent adapters (claude, codex, cursor, copilot, gemini, grok, hermes, droid, antigravity), hookScript, jsonHooksConfig
  lsp/              # LSP server lifecycle: detect, download, helpers, operations, pool, types
  mobileServer/     # Mobile companion: e2ee pairing, discovery, rpc, protocol, deviceStore
  ptyDaemon/        # Persistent terminal daemon: checkpoint, sessionHost, socketServer, lifecycle
  agent.ts          # Main agent orchestration
  agentGenerate.ts  # Content generation helpers
  agentHookServer.ts # Hook server for agent terminal sessions
  agentPermissions.ts # Tool permission allow/deny lists
  agentProcess.ts   # Agent process management
  agentProcessTypes.ts # Agent process type definitions
  agentTypes.ts     # Agent type definitions
  agentUtils.ts     # Agent utility functions
  autoUpdate.ts     # Electron auto-updater integration
  braidMcp.ts       # In-process MCP server for Claude SDK
  claudeConfig.ts   # Claude config (permissions, hooks, skills, MCP)
  claudeConfigMcp.ts # MCP server configuration management
  claudePath.ts     # Claude CLI path detection
  files.ts          # File operations + platform/framework detection
  github.ts         # GitHub API (gh CLI wrapper)
  githubAuth.ts     # GitHub device flow OAuth
  hookInstaller.ts  # Agent hook installation into worktrees
  jira.ts           # Jira integration (acli wrapper)
  lsp-servers.ts    # LSP server definitions/registry
  mcpAuth.ts        # MCP server OAuth authentication
  mcpHealth.ts      # MCP server health checks
  mobileDevice.ts   # Mobile device management
  mobileMcp.ts      # MCP bridge for mobile devices
  notes.ts          # Per-worktree notes persistence
  pty.ts            # Pseudo-terminal (node-pty)
  ptyDaemon/        # Persistent PTY daemon with reattach support
  rgPath.ts         # ripgrep binary path resolution
  scriptDetector.ts # Project script detection (package.json, Makefile, etc.)
  search.ts         # ripgrep-powered file content search + replace
  sessionStorage.ts # Session persistence to ~/Braid/sessions/
  simulator.ts      # iOS/Android simulator control
  storage.ts        # Main-process settings/project persistence
  templates.ts      # Project scaffold templates
  windowCapture.ts  # Screen/window capture for visual context

src/main/lib/       # Utilities: logger, serviceCache, binaryFile, enrichedEnv, errors

src/shared/         # Types shared between main/renderer: search, templates, mobile-protocol, projectName

src/renderer/store/
  sessions/         # Decomposed Zustand store with handlers/ directory
  ui/               # UI preferences: layout, theme, terminal, terminals, settings, apps, helpers slices
  missionControl.ts # Kanban board state
  prCache.ts        # PR status caching (60s refresh)
  projects.ts       # Projects + worktrees store
  rateLimits.ts     # API rate limit tracking
  updater.ts        # Auto-update state
  flash.ts          # Ephemeral toast notifications
  toasts.ts         # Rich toast notifications

src/renderer/components/
  Center/           # ChatView, ChatMessage, ChatHeader, ChatInput, ChatMessageList, ToolCallGroup/, SessionTabBar, BranchBar, StreamingMarkdown, DiffReviewView, CodeReviewView, BigTerminalView, ModelSelector, SlashAutocomplete, MentionAutocomplete, RateLimitBars, WebAppOverlay, ImageLightbox, ElicitationPrompt, ToolPermissionPrompt, ActivityIndicator, TurnFooter, QueuedMessageBanner

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gedeagas/braid](https://github.com/gedeagas/braid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
