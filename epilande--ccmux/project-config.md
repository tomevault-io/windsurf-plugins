---
trigger: always_on
description: ccmux is a CLI tool for tracking AI coding agent sessions running in tmux panes and jumping to the one that needs you. It uses a background daemon that detects agent processes, watches log files, and scans terminal output to derive session state. An interactive TUI shows live session states at a glance.
---

# AGENTS.md

## Project Overview

ccmux is a CLI tool for tracking AI coding agent sessions running in tmux panes and jumping to the one that needs you. It uses a background daemon that detects agent processes, watches log files, and scans terminal output to derive session state. An interactive TUI shows live session states at a glance.

**Built-in agents:** Claude Code, Codex, Cursor, OpenCode, Pi, Gemini CLI, plus custom agent definitions via config.

## Tech Stack

- **Runtime:** Bun 1.x
- **Language:** TypeScript 5.x
- **TUI Framework:** @opentui/solid 0.1.97 (Solid.js-based terminal UI)
- **Reactivity:** Solid.js 1.9
- **File Watching:** native recursive `fs.watch` for the agent log trees (`log-tree-watcher.ts`; chokidar fallback when recursive watching is unavailable), chokidar 4.x for the small flat dirs (markers, Claude subagents)
- **CLI Framework:** Commander 14.x

## Development Guidelines

- Always use `bun` instead of `npm` for all package management and script execution
- Avoid `as any` type casts. Use proper types, generics, or `as unknown as T` only when source types have a genuine gap
- Use `ccmux show` to check session status and see active tmux panes
- You can run `tmux` and `ccmux` commands directly to test features end to end (e.g., `ccmux screen`, `ccmux send`, `ccmux spawn`, `tmux list-panes`, `tmux send-keys`)

### Verifying TUI changes

Typecheck and `bun test` verify code correctness, not rendering correctness. After any change touching TUI components, columns, layout, theming, or the daemon→TUI data path, you MUST launch the picker and/or sidebar and capture the output before declaring the work done. Non-rendering CLI commands (`ccmux show`, `ccmux config get/set`) do not exercise the renderer.

**Always test in a detached, isolated tmux session.** Do not launch the picker in a window/pane inside the user's active session.

Standard workflow:

```bash
# 1. Create a detached session with a forced, known viewport size
tmux new-session -d -s ccmux-verify -x 200 -y 50

# 2. Launch the TUI and let it render (use the same default tmux server
#    so the ccmux daemon can see real sessions; an isolated `-L` socket
#    would render an empty list)
tmux send-keys -t ccmux-verify 'ccmux picker' Enter   # or 'ccmux sidebar'
sleep 3
tmux capture-pane -t ccmux-verify -p | head -40

# 3. Resize / re-capture to test responsive breakpoints if relevant
tmux resize-window -t ccmux-verify -x 60 -y 30
sleep 1
tmux capture-pane -t ccmux-verify -p | head -20

# 4. Tear it all down
tmux kill-session -t ccmux-verify
```

Verify the specific area you changed: column alignment, active indicator (`▎`), agent colors, row collapse vs subtitle, sidebar layout, etc. If you can't reach the path you changed (e.g., needs a session in a specific state that doesn't currently exist), say so explicitly rather than claiming success.

### Do Not

- Do not add new daemon modules without wiring them into `src/daemon/index.ts`
- Do not modify the SSE event protocol without updating both `src/daemon/server.ts` and `src/tui/utils/sse.ts`
- Do not add built-in agents without defining `terminalRules` for pane-tracked detection
- Do not register new `HookAdapter`s in `src/commands/setup.ts` and `src/daemon/index.ts` separately. Add them to `createBuiltinHookAdapters()` in `src/daemon/adapters/index.ts` — both call sites go through it

## Commands

```bash
# Run CLI
ccmux                    # Run the CLI (via ./bin/ccmux or bun link)

# Type checking and testing
bun run typecheck        # TypeScript type check (tsc --noEmit)
bun test                 # Run all tests
bun test src/daemon/parser.test.ts  # Run single test file

# Build
bun run build            # Bundle to dist/index.js

# Performance profiling
CCMUX_PERF=1 ccmux picker 2>/tmp/perf.log  # Startup waterfall + runtime stats
```

Full CLI reference: `ccmux --help` or see README.md.

## Architecture

### Daemon

The daemon (`src/daemon/`) detects agent processes in tmux panes, watches log files, and scans terminal output. The key modules to orient by:

- **State Reconciler** (`state-reconciler.ts`) - Core loop reconciling native (hook-tracked) and pane-tracked sessions with live tmux state. The `background` tracking mode is owned solely by the Background Source and is excluded from every reconciler arm.
- **Cascade Evaluator** (`cascade-evaluator.ts`) - Pure freshest-wins-with-tiebreak fold over `CascadeSource[]` (marker, log, terminal). Tie-break `marker > log > terminal`; `upgradeOnly` sources can lift status to `waiting` but never downgrade. Used by both native and pane-tracked paths.
- **Status Machine** (`status-machine.ts`) - Derives session state from JSONL log entries.
- **Terminal Detector** (`terminal-detector.ts`) - Pattern-based status detection from pane content (agents without log parsing).
- **Binder** (`binder/`) - All session-to-pane matching policy: marker claims settle first (authoritative, re-asserted every scan), then heuristic assignment gated by direction skew, a 600s cap, and an ambiguity refusal.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [epilande/ccmux](https://github.com/epilande/ccmux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
