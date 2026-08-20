---
trigger: always_on
description: - When the user asks a question, answer it first before making edits or running implementation commands.
---

# Development Rules

## Conversational Style

- When the user asks a question, answer it first before making edits or running implementation commands.
- When responding to user feedback or an analysis, explicitly say whether you agree or disagree before saying what you changed.
- Challenge me and push back and play devils advocate when i want to add implement something that has risks or for a new feature.

This file provides guidance to models when working with code in this repository.

## GOLDEN RULE: Docs & AGENTS.md

Update corresponding Docs in [CrewCode Docs](/docs/), and [AGENTS.md](/AGENTS.md/) when major changes were made and or every time i add a feature. Create or update docs file for it

## What is CrewCode?

CrewCode is a desktop ACE (Agent Coding Environment) GUI built with Electron + React + TypeScript. It lets developers run a *crew* of AI coding agents (Claude Code, Codex, OpenCode, etc.) in parallel across local git worktrees, each in its own workspace with a chat thread, embedded terminal panes, and a code/markdown editor — all in one frameless native-feeling window.

## Commands

```bash
npm run dev        # Start dev server (Vite renderer at localhost:5173) + Electron main process
npm run build      # Build all three processes (main, preload, renderer) via electron-vite
npm run preview    # Preview the production build
npm run typecheck  # Run tsc --noEmit across all tsconfigs
npm run ship -- "feat: msg"  # Stage + commit + push current branch to origin
npm run release    # Verify, bump patch version, tag, push -> triggers CI release build
```

> `npm run dev` uses `env -u ELECTRON_RUN_AS_NODE` to prevent Electron's Node.js mode from interfering.

## Architecture

This is a standard **electron-vite** three-process project:

| Process      | Entry                  | Purpose                                                                            |
| ------------ | ---------------------- | ---------------------------------------------------------------------------------- |
| **Main**     | `src/main/index.ts`    | Creates `BrowserWindow`, handles IPC for window controls (minimize/maximize/close) |
| **Preload**  | `src/preload/index.ts` | Exposes `window.electronAPI` to renderer via `contextBridge`                       |
| **Renderer** | `src/renderer/src/`    | React SPA — the entire UI                                                          |

Built output lands in `out/` (gitignored). In dev, the renderer runs at `ELECTRON_RENDERER_URL` (Vite dev server); in production it loads `out/renderer/index.html`.

### Renderer structure

```
src/renderer/src/
├── App.tsx               # Root — all top-level state lives here
├── main.tsx              # React entry point
├── types/index.ts        # All shared types (Tab, Message, Workspace, TermSession, etc.)
├── hooks/
│   └── useTweaks.ts      # Generic key-value state hook for TweakConfig
├── data/                 # Static mock data (workspaces, termSessions, codeFiles, commands)
├── styles/
│   ├── colors_and_type.css  # Full CSS token set — imported globally
│   └── styles.css           # Layout and component styles
└── components/
    ├── ui/               # WindowTabs, Icon, StatusPill
    ├── thread/           # ChatHeader, Messages, Sessions, WorkLog
    ├── composer/         # Composer, ModelRow, ModeSegment
    ├── terminal/         # TermColumn, TermPane
    ├── editor/           # CodeEditor, FileTree, MarkdownEditor
    ├── workspaces/       # WorkspacesDrawer, WorkspaceDock, WorkspaceRow
    ├── CommandPalette.tsx
    └── TweaksPanel.tsx
```

`App.tsx` owns all state and passes it down. There is no global state manager — everything is React `useState`.

## Worktree Safety

Always use the primary working directory (the worktree) for all file reads and edits. Never follow absolute paths from subagent results that point to the main repo.

## GOVERNING DOCTRINE: Execution Custody

Binding on every privileged surface in this repository, current and future. Full
rationale and implementation map in `docs/execution-custody.md`.

Granting authority is decided at the gates in `docs/security-model.md`. This
doctrine governs the other half of the lifecycle: **withdrawing authority once it
has already been granted.**

When **authority / identity / scope / provenance / execution custody** becomes
unknown, stale, contradictory, or changes unexpectedly:

```
-> refuse new privileged actions on the affected scope
-> contain or terminate owned execution where safe
-> preserve evidence and current workspace state
-> report the exact failed invariant and affected scope
-> require explicit human reauthorization before resuming
```

Never, under any circumstance, infer a successful outcome from the absence of a
failure signal:

```
silence               != success
timeout               != success
lost telemetry        != success
missing process state != success
clean Git state       != behavioral correctness
```

Rules for new code:

- An operation whose outcome was never observed is recorded as `interrupted` or
  `halted`. It is never back-filled as complete, and never on restart.
- Long-lived executions carry a persisted custody record. Process-local runtime

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OnPoint-Dev-Tools/crewcode](https://github.com/OnPoint-Dev-Tools/crewcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
