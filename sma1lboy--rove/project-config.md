---
trigger: always_on
description: Rove is a local-first terminal UI for running many AI coding sessions at once — Conductor's multi-task shape (task sidebar, workspace chat/files tabs, file tree, embedded terminal, status bar) made terminal-native with git worktrees and local engine processes.
---

# Rove (repository/package compatibility name: kobe)

## Project at a glance

Rove is a local-first terminal UI for running many AI coding sessions at once — Conductor's multi-task shape (task sidebar, workspace chat/files tabs, file tree, embedded terminal, status bar) made terminal-native with git worktrees and local engine processes.

The isolation unit for a managed Task is:

```text
Managed task = git worktree + branch + terminal tabs
```

Project-main Tasks reuse a saved repository checkout, and directory Tasks reuse
a user-owned directory; neither owns a Rove-created worktree or branch.

The TUI is the product; engine adapters are execution backends (Claude Code is the default, Codex lives behind the same engine-owned contract). This file is a lean operator manual — **boundaries and orientation only**. Mechanics live in `docs/`; the current version + shipped behavior live in [`packages/kobe/package.json`](./packages/kobe/package.json) and [`packages/kobe/CHANGELOG.md`](./packages/kobe/CHANGELOG.md). Don't duplicate those here.

**Read in order before doing anything:**
1. [`HANDOFF.md`](./HANDOFF.md) — freshest handoff, current risks, open follow-ups. Local + gitignored; absent on a fresh clone is fine, just skip it.
2. [`docs/DESIGN.md`](./docs/DESIGN.md) — design philosophy, decisions, tech-stack lock-in.
3. [`docs/ARCHITECTURE.md`](./docs/ARCHITECTURE.md) — source-tree map, ownership boundaries, and the `refs/` reference projects (§7).
4. [`docs/HARNESS.md`](./docs/HARNESS.md) — agent self-test contract. **Load-bearing.**
5. [`docs/KEYBINDINGS.md`](./docs/KEYBINDINGS.md) — pane-scope rules; read before adding/moving any chord.
6. [`packages/kobe/CHANGELOG.md`](./packages/kobe/CHANGELOG.md) — shipped behavior + release-note style.

The docs are the source of truth. **If docs and implementation disagree, surface the mismatch before widening scope.**

## Orientation

- **Monorepo (Bun workspaces), source under `packages/`:** `kobe/` (the TUI/CLI, published canonically as `@sma1lboy/rove` and compatibly as `@sma1lboy/kobe`), `kobe-daemon/` (daemon server + protocol + socket client), `kobe-harness/` (the `/harness` capture page + PTY sidecar), `branding/` (Remotion pipeline), `kobe-docs/` (public docs site, Fumadocs on Next.js, static export; content synced from `docs/`). Unqualified `src/…`/`test/…` paths in docs are relative to `packages/kobe/`. Full source-tree map: [`docs/ARCHITECTURE.md`](./docs/ARCHITECTURE.md).
- **Three test runners, and picking the wrong one looks like a broken environment.** `test/render/**` runs under bun's own runner (`bun test test/render`) because OpenTUI needs bun; `test/daemon/**` needs `KOBE_INCLUDE_SOCKET=1` (`bun run test:socket`) and without it vitest prints "No test files found" and exits 1 — a SILENT skip that reads like a missing file, not a wrong command; **everything else runs under vitest** (`bun run test:fast`, or `bun x vitest run <file>` for one file). Running a vitest file with `bun test` fails on vitest-only APIs — `vi.hoisted is not a function` is the usual signature, and it reads like a missing dependency rather than the wrong command. If a test "can't run", check the runner before concluding anything about the environment.
- **Run scripts** via `bun --filter @sma1lboy/rove <script>` or `cd packages/kobe && bun <script>`. Two dev flavours: `dev` (real engines, **production** Rove state) and `dev:sandbox` (real engines + your real `HOME`, throwaway Rove state under `packages/kobe/.dev-sandbox/home`) — use the sandbox so you never touch the real `~/.rove/tasks.json`.
- **Tech stack is locked:** TypeScript + `@opentui/core` + `@opentui/react` + React 19 + Bun. Do not re-litigate. React is the only UI; orchestrator/client reactivity is framework-free observable state — don't add UI-framework state primitives to the core.
- **UI development is OpenTUI-first, with one visual ground truth.** Develop the real `packages/kobe/src/tui-react/**` surface through `dev:sandbox`. For every agent-driven visual iteration, screenshot, and UI acceptance check, the **only** ground-truth path is a fixed-viewport browser `/harness` → xterm.js → PTY sidecar → real OpenTUI. Do not use local Terminal screenshots, render-test output, or alternate mocks as visual substitutes. When a bug is about live state rather than layout ("the badge never cleared"), drive a REAL engine down the same path — keys through the browser's xterm, state via `rove api inspect`. The shortcuts that silently measure nothing are catalogued in [`docs/HARNESS.md`](./docs/HARNESS.md).
- **Language:** respond in whatever language the user writes in. Don't assume their name — let them introduce themselves.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sma1lboy/rove](https://github.com/Sma1lboy/rove) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
