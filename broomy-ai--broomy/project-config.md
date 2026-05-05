---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Setup

```bash
pnpm install         # Install dependencies (use pnpm, not npm/yarn)
```

## Commands

```bash
pnpm dev             # Development with hot reload (renderer only; restart for main/preload changes)
pnpm build           # Build without packaging
pnpm dist            # Build and package for macOS
pnpm storybook       # Storybook dev server on port 6006
pnpm storybook:test  # Visual regression test (screenshot + pixel-diff + report)
pnpm storybook:update-refs  # Accept current screenshots as reference baseline
```

## Shell Commands

**Never use `${}` syntax (shell parameter expansion) in Bash tool calls.** This includes `$(...)` subshells and `${VAR}` variable expansions. These trigger manual approval prompts. Instead, use plain variable references like `$VAR`, pipe chains, or break commands into multiple sequential tool calls.

**Don't run tests or checks manually** — use `/validate` instead. It runs all checks in the right order and fixes failures automatically.

## Troubleshooting

`pnpm dev` runs preflight checks automatically and fixes common issues (missing Electron binary, native module problems). If preflight can't fix something, it tells you what to do.

**Nuclear option**: `rm -rf node_modules && pnpm install`

**Important**: This project enforces pnpm via a preinstall script. Do not use npm or yarn.

## Architecture

Broomy is an Electron + React desktop app for managing multiple AI coding agent sessions across different repositories. See `docs/architecture.md` for the full technical guide.

### Process Structure

- **Main process** (`src/main/index.ts`): All IPC handlers -- PTY management (node-pty), git operations (simple-git), filesystem I/O, GitHub CLI wrappers, config persistence, window lifecycle. Every handler checks `isE2ETest` and returns mock data during tests.
- **Preload** (`src/preload/index.ts`): Context bridge + type definitions. Exposes `window.pty`, `window.fs`, `window.git`, `window.gh`, `window.config`, `window.profiles`, `window.shell`, `window.repos`, `window.app`, `window.menu`, `window.dialog`.
- **Renderer** (`src/renderer/`): React UI with Zustand state management and Tailwind CSS.

### Key Renderer Organization

- **Panels** (`panels/`): Each UI panel is a self-contained module owning its components, hooks, and utils. See `panels/README.md` for conventions.
  - `panels/system/` -- Panel registry infrastructure (types, registry, context)
  - `panels/sidebar/` -- Session list sidebar (SessionList, SessionCard)
  - `panels/explorer/` -- Explorer with tabbed sub-panels (`tabs/files`, `tabs/source-control`, `tabs/search`, `tabs/recent`, `tabs/review`)
  - `panels/fileViewer/` -- File viewer with plugin-based `viewers/` registry (Monaco, Image, Markdown, Webview, diff viewers) and `hooks/` for file loading/watching/navigation
  - `panels/agent/` -- Terminal emulator (`Terminal.tsx`, `TabbedTerminal.tsx`) with `hooks/` (PTY setup, keyboard) and `utils/` (stripAnsi, activity detection, buffer registry). **Never unmounts on session switch.**
  - `panels/settings/` -- Agent and repo configuration overlay
  - `panels/tutorial/` -- Getting-started guide
- **Features** (`features/`): Cross-cutting domain logic used by multiple panels.
  - `features/git/` -- Branch status, git status normalization, explorer helpers, git polling, plan detection
  - `features/sessions/` -- New session dialog wizard, session lifecycle hooks
  - `features/profiles/` -- Profile chip and dropdown
  - `features/commands/` -- Commands config loading, condition evaluation, action execution
- **Shared** (`shared/`): Generic components, hooks, and utils used by 2+ panels/features.
  - `shared/components/` -- ErrorBoundary, PanelErrorBoundary, ErrorBanner, ActionButtons, modals
  - `shared/hooks/` -- Layout keyboard/resize, app callbacks, session keyboard, update state
  - `shared/utils/` -- File navigation, focus helpers, slugify, text detection, markdown components
- **Layout** (`layout/`): Top-level layout shell -- Layout, LayoutContentArea, LayoutToolbar, Divider, VersionIndicator
- **Stores** (`store/`): Zustand stores -- `sessions.ts`, `agents.ts`, `repos.ts`, `profiles.ts`, `errors.ts`, `tutorial.ts`. Session store actions split into `sessionCoreActions.ts`, `sessionPanelActions.ts`, `sessionBranchActions.ts`, `sessionTerminalTabs.ts`.

### Agent Activity Detection

Agent status is detected by time-based heuristics in `Terminal.tsx`. The detection logic:
- **Warmup**: Ignores the first 5 seconds after terminal creation
- **Input suppression**: Pauses detection for 200ms after user input or window interaction
- **Working**: Set immediately when terminal data arrives (if not paused)
- **Idle**: Set after 1 second of no terminal output, with a 300ms debounce for store updates

When a session transitions from working to idle (after at least 3 seconds of working), it's marked as `isUnread` to alert the user.

### Data Persistence

Config files at `~/.broomy/profiles/<profileId>/`:
- `config.json` (production) / `config.dev.json` (development)
- Contains agents, sessions with panel visibility and layout sizes, repos, toolbar panel order


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Broomy-AI/broomy](https://github.com/Broomy-AI/broomy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
