---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# ClaudeUI

A desktop GUI for Claude Code sessions, built with Electron. Features include multi-session chat, integrated git UI, terminal emulator, automation scheduling, remote web access, voice input, plugin system, and usage analytics.

## Tech Stack

- **Electron** with `electron-vite` (react-ts template)
- **React 19** + **TypeScript**
- **Tailwind CSS v4** via `@tailwindcss/vite` plugin (no postcss/tailwind config files)
- **Zustand** for state management
- **react-markdown** + **remark-gfm** for rendering
- **@anthropic-ai/claude-agent-sdk** (0.2.97) for Claude integration
- **simple-git** for git operations
- **node-pty** + **@xterm/xterm** for terminal emulator
- **mermaid** for diagram rendering
- **cron-parser** for automation scheduling
- **Prism.js** for syntax highlighting in diffs
- **Package manager: bun** (not npm/yarn)

## Commands

- `bun run dev` — start in development mode with hot reload
- `bun run build` — typecheck + build
- `bun run build:mac` — build macOS distributable
- `bun run build:win` — build Windows distributable
- `bun run test` — run Vitest tests
- `bun run test:watch` — run tests in watch mode
- `bun run typecheck` — run TypeScript checks (node + web)
- `bun run lint` — ESLint
- `bun run format` — Prettier
- `bun patch/apply-all.mjs` — apply SDK patches (also runs automatically via `postinstall`)

## Project Structure

```
src/
  shared/
    types.ts               — All shared TypeScript types (ContentBlock, ChatMessage, ClaudeAPI, etc.)
    remote-protocol.ts     — WebSocket message types for remote access
    e2e-crypto.ts          — AES-256-GCM E2E encryption (isomorphic Node + browser)
  main/
    index.ts               — Electron BrowserWindow setup, app lifecycle, service wiring
    ipc/
      session.ipc.ts       — Core IPC: sessions, git, config, MCP, usage, worktrees, voice, proxy
      terminal.ipc.ts      — PTY create/write/resize/kill
      automation.ipc.ts    — Automation CRUD + run management
      remote-handlers.ts   — WebSocket dispatch bridge (same handlers as session.ipc)
    services/              — 29 service modules (see Services section below)
  preload/
    index.ts               — Context bridge (ClaudeAPI → window.api, 280+ lines)
    plugin-preload.ts      — Plugin sandbox bridge (window.pluginApi)
    log-viewer-preload.ts  — Log viewer window bridge
  renderer/src/
    stores/
      session-store.ts     — Primary Zustand store (sessions, settings, git, terminals, etc.)
      automation-store.ts  — Automation scheduling state
    hooks/
      useClaudeEvents.ts   — Master IPC event listener → store actions
      useAutomationEvents.ts — Automation IPC listeners
      useGitWatcher.ts     — Git repo detection + status polling
      useSlashMenu.ts      — Slash command (/) autocomplete
      useFileMention.ts    — File mention (@path) autocomplete with directory browsing
      useTerminalColdCleanup.ts — Orphaned terminal cleanup (10-min timeout)
      useGutterDragSelection.ts — Line selection in diffs
      useTextSelectionComment.ts — Inline comment placement in diffs
      useIsMobile.ts       — Responsive breakpoint detection
    components/
      SessionView.tsx      — Root layout (sidebar + main + right panel + terminal)
      Sidebar.tsx          — Session list, directory browser, pinning, custom titles
      WelcomeScreen.tsx    — Initial folder picker
      SettingsDialog.tsx   — Full settings UI (theme, fonts, diff, sandbox, proxy, voice)
      PermissionsDialog.tsx — Claude permission rule management
      McpDialog.tsx        — MCP server configuration
      SkillsDialog.tsx     — Available skills listing
      RemoteAccessModal.tsx — Remote web client connection
      TodoWidget.tsx       — Floating task list overlay
      TaskDetailPanel.tsx  — Task/subagent detail sidebar
      TeamsView.tsx        — Multi-agent team monitoring
      WindowControls.tsx   — macOS traffic light buttons
      WorktreesModal.tsx   — Git worktree management
      QuitWorktreeModal.tsx — Worktree cleanup on quit
      WorktreeCleanupModal.tsx — Orphaned worktree cleanup
      chat/                — 24 components (ChatPanel, InputBox, MessageBubble, ToolCallBlock,
                             StreamingText, FloatingApproval, FloatingError, ThinkingBlock,
                             SubagentMessages, AgentTabBar, SlashCommandMenu, FileMentionMenu,
                             FileAttachmentBar, MermaidDiagram, MarkdownRenderer, CodeView,
                             ExitPlanModeCard, TaskCard, TodoToolBlock, TerminalView,
                             AskUserQuestionBlock, BtwCard, PermissionSuggestions,
                             SandboxViolationToast)
      git/                 — 11 components (GitPanel, GitFileTree, GitFileDiffView, GitCommitBox,
                             GitBranchDropdown, GitBranchPill, GitChangesPill, WorktreePill,
                             DiffCommentBadge, DiffCommentWidget, ReviewBar)
      plan/                — 4 components (PlanReviewPanel, PlanReviewBar, PlanCommentBadge,
                             PlanCommentWidget)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wellofspirit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
