---
trigger: always_on
description: Kirodex is a native macOS desktop app for managing AI coding agents via the Agent Client Protocol (ACP). It features a chat interface, task management, diff viewer, integrated terminal, git operations, and a settings panel. Built with Tauri v2 (Rust backend) and React 19 (TypeScript frontend).
---

# CLAUDE.md — Kirodex

## Project overview

Kirodex is a native macOS desktop app for managing AI coding agents via the Agent Client Protocol (ACP). It features a chat interface, task management, diff viewer, integrated terminal, git operations, and a settings panel. Built with Tauri v2 (Rust backend) and React 19 (TypeScript frontend).

## Tech stack

- **Desktop framework**: Tauri v2 (Rust backend, WebView frontend)
- **Backend**: Rust 2021 edition
- **Frontend**: React 19, TypeScript 5, Vite 6
- **Styling**: Tailwind CSS 4 (utility-first, dark theme)
- **UI components**: Radix UI primitives, Tabler icons (`@tabler/icons-react`)
- **State management**: Zustand 5 (stores in `src/renderer/stores/`)
- **Markdown**: react-markdown + remark-gfm
- **Virtualization**: @tanstack/react-virtual
- **Diffing**: diff + @pierre/diffs
- **Terminal**: xterm + @xterm/addon-fit + portable-pty (Rust)
- **Code highlighting**: Shiki
- **Build**: Vite (renderer), Cargo (Rust backend), bun as package manager
- **Protocol**: agent-client-protocol crate for ACP subprocess management
- **Rust crates**: git2 (libgit2 bindings), thiserror (error types), which (binary detection), serde_yaml (YAML parsing), confy (config persistence)

## Project structure

```
src/
├── renderer/                # React frontend
│   ├── main.tsx             # React entry
│   ├── App.tsx              # Root layout
│   ├── types/index.ts       # Shared types (TaskStatus, AgentTask, etc.)
│   ├── lib/
│   │   ├── ipc.ts           # Tauri invoke/listen wrappers
│   │   ├── timeline.ts      # Timeline rendering logic
│   │   └── utils.ts         # cn() helper
│   ├── hooks/
│   │   └── useSlashAction.ts # Client-side slash command handler
│   ├── stores/
│   │   ├── taskStore.ts     # Tasks, streaming, connection state
│   │   ├── settingsStore.ts # Agent profiles, models, appearance
│   │   ├── kiroStore.ts     # .kiro/ config state
│   │   ├── diffStore.ts     # Diff viewer state
│   │   └── debugStore.ts    # Debug panel state
│   └── components/
│       ├── ui/              # Radix-based primitives (button, input, dialog, etc.)
│       ├── chat/            # ChatPanel, MessageList, ChatInput, SlashPanels, etc.
│       ├── sidebar/         # TaskSidebar, KiroConfigPanel
│       ├── code/            # CodePanel, DiffViewer
│       ├── dashboard/       # Dashboard, TaskCard
│       ├── settings/        # SettingsPanel
│       ├── diff/            # DiffPanel
│       ├── debug/           # DebugPanel
│       ├── task/            # NewProjectSheet
│       ├── AppHeader.tsx
│       ├── ErrorBoundary.tsx
│       └── Playground.tsx
src-tauri/
├── src/
│   ├── main.rs              # Entry point
│   ├── lib.rs               # Tauri app setup, command registration, window events
│   └── commands/
│       ├── acp.rs           # ACP protocol (kiro-cli subprocess, ~42KB)
│       ├── error.rs         # Shared AppError type (thiserror)
│       ├── pty.rs           # Terminal emulation (portable-pty)
│       ├── git.rs           # Git operations via git2 (libgit2)
│       ├── settings.rs      # Config persistence via confy
│       ├── fs_ops.rs        # File ops, kiro-cli detection (which crate)
│       └── kiro_config.rs   # .kiro/ config discovery (serde_yaml for frontmatter)
├── Cargo.toml
├── tauri.conf.json
└── capabilities/            # Tauri v2 permission capabilities
```

## Commands

```bash
bun run dev           # Start dev (Vite + Tauri)
bun run build         # Production build (.app + .dmg)
bun run check:ts      # TypeScript type check
bun run check:rust    # Rust type check
bun run test:rust     # Run Rust tests
bun run clean         # Remove build artifacts
```

## Architecture decisions

- **Tauri IPC**: All frontend↔backend communication uses `invoke()` for commands and `listen()` for events. No direct Node.js APIs.
- **ACP on dedicated OS threads**: The ACP Rust SDK uses `!Send` futures, so each connection runs on a dedicated OS thread with a single-threaded tokio runtime + `LocalSet`. Communication with the Tauri async runtime happens via `mpsc` channels.
- **Permission handling**: Permission requests from ACP go through a `oneshot` channel. The permission handler runs on the Tauri async runtime and accesses managed state via `app.try_state::<AcpState>()`, not a cloned copy.
- **State**: Zustand stores are the single source of truth. No Redux, no Context for global state.
- **Styling**: Tailwind utility classes only. No custom CSS files for components. Theme tokens in `src/tailwind.css`.
- **Components**: Radix UI primitives with `class-variance-authority` for variants, `clsx` + `tailwind-merge` via `cn()` helper.
- **Path aliases**: `@/*` maps to `./src/renderer/*` (configured in tsconfig.json and vite.config.ts).

## Conventions

- Use `const` arrow functions for components and handlers
- Prefix event handlers with `handle` (e.g., `handleClick`, `handleKeyDown`)
- Prefix boolean variables with verbs (`isLoading`, `hasError`, `canSubmit`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thabti/kirodex](https://github.com/thabti/kirodex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
