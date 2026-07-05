---
trigger: always_on
description: kobe is a local-first terminal UI for running many AI coding sessions at once — Conductor's multi-task shape (task sidebar, workspace chat/files tabs, file tree, embedded terminal, status bar) made terminal-native with git worktrees and local engine processes.
---

# kobe (codename, rename later)

## Project at a glance

kobe is a local-first terminal UI for running many AI coding sessions at once — Conductor's multi-task shape (task sidebar, workspace chat/files tabs, file tree, embedded terminal, status bar) made terminal-native with git worktrees and local engine processes.

The product unit is:

```text
Task = git worktree + engine session + branch
```

The TUI is the product; engine adapters are execution backends (Claude Code is the default, Codex lives behind the same engine-owned contract). This file is a lean operator manual — **boundaries and orientation only**. Mechanics live in `docs/`; the current version + shipped behavior live in [`packages/kobe/package.json`](./packages/kobe/package.json) and [`packages/kobe/CHANGELOG.md`](./packages/kobe/CHANGELOG.md). Don't duplicate those here.

**Read in order before doing anything:**
1. [`HANDOFF.md`](./HANDOFF.md) — freshest handoff, current risks, open follow-ups. Local + gitignored; absent on a fresh clone is fine, just skip it.
2. [`docs/DESIGN.md`](./docs/DESIGN.md) — design philosophy, decisions, tech-stack lock-in.
3. [`docs/ARCHITECTURE.md`](./docs/ARCHITECTURE.md) — source-tree map, ownership boundaries, and the `refs/` reference projects (§2).
4. [`docs/PLAN.md`](./docs/PLAN.md) — phase/wave plan + gate history (**phase status lives here, not in this file**).
5. [`docs/HARNESS.md`](./docs/HARNESS.md) — agent self-test contract. **Load-bearing.**
6. [`docs/KEYBINDINGS.md`](./docs/KEYBINDINGS.md) — pane-scope rules; read before adding/moving any chord.
7. [`packages/kobe/CHANGELOG.md`](./packages/kobe/CHANGELOG.md) — shipped behavior + release-note style.

The docs are the source of truth. **If docs and implementation disagree, surface the mismatch before widening scope.**

## Orientation

- **Monorepo (Bun workspaces), source under `packages/`:** `kobe/` (the TUI/CLI, published as `@sma1lboy/kobe`), `kobe-daemon/` (daemon server + protocol + socket client + daemon-hosted web transport), `kobe-web/` (the browser dashboard SPA + PTY sidecar), `branding/` (Remotion pipeline). Unqualified `src/…`/`test/…` paths in docs are relative to `packages/kobe/`. Full source-tree map: [`docs/ARCHITECTURE.md`](./docs/ARCHITECTURE.md).
- **Run scripts** via `bun --filter @sma1lboy/kobe <script>` or `cd packages/kobe && bun <script>`. Two dev flavours: `dev` (real engines, **production** `~/.kobe`) and `dev:sandbox` (real engines, throwaway `packages/kobe/.dev-sandbox/home` + `KOBE_TMUX_SOCKET=kobe-sandbox`) — use the sandbox so you never touch the real `~/.kobe/tasks.json`.
- **Tech stack is locked:** TypeScript + `@opentui/core` + `@opentui/solid` + Solid.js + Bun. Do not re-litigate.
- **Language:** respond in whatever language the user writes in. Don't assume their name — let them introduce themselves.
- **Daemon** is a long-lived background process, refcounted on attached GUIs (mechanics: [`docs/design/daemon.md`](./docs/design/daemon.md)). Boundaries: in-tmux helper panes subscribe with `role: "pane"`; attached TUI clients and open browser SSE streams hold GUI lifetime; daemon shutdown never touches tmux; read `<KOBE_HOME>/.kobe/daemon.log` first when debugging; **after editing daemon/orchestrator/engine code, `kobe daemon restart`** — Bun doesn't hot-reload.
- **Per-repo init:** a repo can ship `.kobe/init.sh` (runs before the engine, in the worktree) + `.kobe/init-prompt.md` (the engine's first message); repo files win over the per-user state.json override. Mechanics: [`src/state/repo-init.ts`](./packages/kobe/src/state/repo-init.ts).
- **Reference repos** (`refs/`, gitignored, **read-only**): clone before development — what each is for + when to consult lives in [`docs/ARCHITECTURE.md`](./docs/ARCHITECTURE.md) §2.

```bash
mkdir -p refs && cd refs
ln -s /Users/jacksonc/i/agent-deck agent-deck   # if you have it locally
git clone --depth 1 https://github.com/winfunc/opcode.git
git clone --depth 1 https://github.com/tanbiralam/claude-code.git
git clone --depth 1 https://github.com/sirmalloc/ccstatusline.git
git clone --depth 1 https://github.com/openai/codex.git
git clone --depth 1 https://github.com/friuns2/codexui.git
git clone --depth 1 https://github.com/warpdotdev/warp.git
# conductor is image-only — see docs/DESIGN.md §1
```

## Work tracking — local only

No Linear. Backlog/open issues live in the daemon-owned issue store (web Issues page or `kobe api issue-*`, see [`docs/WORK-TRACKING.md`](./docs/WORK-TRACKING.md)); shipped behavior in [`packages/kobe/CHANGELOG.md`](./packages/kobe/CHANGELOG.md) (one Changeset per change, see [`docs/RELEASING.md`](./docs/RELEASING.md)); current risks/follow-ups in [`HANDOFF.md`](./HANDOFF.md); durable design decisions as Markdown in `docs/`. If a requirement needs external tracking, surface it first instead of filing it automatically.

## Hard rules (non-negotiable)

### PR-only mainline (2026-07-03)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sma1lboy/kobe](https://github.com/Sma1lboy/kobe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
