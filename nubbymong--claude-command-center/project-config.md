---
trigger: always_on
description: Canonical instructions for any AI agent or contributor working in this repo. This
---

# AI Code Conductor — agent & contributor brief

Canonical instructions for any AI agent or contributor working in this repo. This
is the cross-tool standard file (read by Claude Code via `@AGENTS.md` in
`CLAUDE.md`, and directly by Codex, Cursor, Copilot, and others). Read it before
making changes.

Multi-session Claude Code terminal orchestrator built with Electron 42 + React 19
+ TypeScript.

## Session isolation — do this FIRST

Several agents run against this repo simultaneously. **One session = one worktree
= one branch.** Before changing anything, claim your own:

```bash
node scripts/session-guard.mjs claim --base beta   # or: adopt (already in one)
```

Work only in the directory it prints, and prefer `git -C "<that dir>" …` over
relying on the current directory. Never work in the primary checkout or another
session's worktree — their branch can change under you and they may hold
uncommitted work. A `PreToolUse` hook denies writes and mutating git outside the
worktree you own; `CCC_SESSION_GUARD=off` is the escape hatch. See
`docs/session-isolation.md` and ADR-012.

## Ticket creation & premise review — policy

**Every repo change starts from a GitHub issue, and every issue carries a premise
review.** Two standing gates bracket a change: a *premise* review at creation, and
an *adversarial* review before merge (ADR-009). This is the first one.

Before you file an issue — whether you are a human or an agent — state its
**premise** and check it holds:

- **The problem, and the evidence it is real.** Not "X would be nice" but "X is
  broken/absent, here is where (`file:line`), here is what happens." Ground it in
  the code as it is now, not as you remember it.
- **Why it is still open.** This repo moves fast; a surprising amount of proposed
  work is already shipped or half-shipped. Check recent merges before asserting a
  problem exists — a STALE premise is the most common and most wasteful defect.
- **Why now / what it blocks.** Enough for triage to place it on a release line.

An agent that files an issue **must** include a short premise-assessment section in
the body (problem · evidence · still-open · why-now). An issue without one is
incomplete and should be sent back, exactly as a security-sensitive PR without an
adversarial pass would be.

For the existing backlog that predates this policy, the `/LoopReady` skill runs the
premise review in bulk (a cheap Fable fan-out) and labels each ticket
`loop-ready` / `loop-needs-human`; `/StartLoop` re-checks the premise once more
before spending real model budget executing it. See
`.claude/skills/LoopReady/SKILL.md` and `.claude/skills/StartLoop/SKILL.md`. These
are AI Code Conductor-only; they encode this repo's `beta` / session-guard /
ADR-009 / label conventions and are not the aai-core loop skills.

## Build & Run

```bash
npm run dev          # Development with HMR (prefer the `ccc` launcher — see below)
npm run build        # Production build (electron-vite)
npm run typecheck    # tsc --noEmit
npm run test:unit    # vitest
npm run test:e2e     # playwright
npm run test         # both
```

- Prefer the `ccc` launcher for dev — it isolates dev data from prod and cleans
  up all dev processes on exit. See `docs/dev-alongside-prod.md`.
- **Never run `npm install` in a worktree — use `npm ci` (#226).** `npm install`
  rewrites the tree and can leave it looking fully installed while Electron's own
  binary is absent: `node_modules/electron/` exists, `package.json` is satisfied
  and `npm ls` is clean, but `path.txt` and `dist/electron.exe` are gone. The
  repo's `postinstall` only rebuilds the two native addons, so nothing replaces
  them. electron-vite then dies with an opaque `Error: Electron uninstall`, the
  launcher window closes instantly, and the real message is only in
  `dev-logs/ccc-dev-*.log`. `predev` (`scripts/preflight-electron.mjs`) now
  detects this and re-runs Electron's installer before dev starts, but the rule
  stands: `npm ci`, and it must be run in the worktree you are working in.

## Architecture

- **Main process** (`src/main/`): Electron main, PTY management (node-pty), IPC handlers, config persistence, statusline, vision MCP server, cloud agents, tokenomics
- **Renderer** (`src/renderer/`): React 19 SPA with Zustand stores, xterm.js terminals, Tailwind CSS v4
- **Preload** (`src/preload/`): IPC bridge - all renderer↔main communication goes through typed channels
- **Shared** (`src/shared/`): Types and IPC channel constants used by both processes

### Key patterns

- IPC handlers are in `src/main/ipc/` - one file per domain (pty, config, logs, etc.)
- Config persistence via `src/main/config-manager.ts` - JSON files in a user-selected resources directory
- Stores in `src/renderer/stores/` - Zustand, hydrated from config on startup
- Terminal rendering via xterm.js with WebGL addon
- SSH sessions use node-pty to spawn ssh.exe, with automated setup scripts for statusline/vision

## Coding Conventions

- No default exports (except React components that are the sole export of their file)
- Tailwind v4 with `@theme` in `src/renderer/styles.css` - no tailwind.config file
- Catppuccin Mocha color palette (base, mantle, crust, surface0-2, overlay0-2, subtext0-1, text, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nubbymong/claude-command-center](https://github.com/nubbymong/claude-command-center) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
