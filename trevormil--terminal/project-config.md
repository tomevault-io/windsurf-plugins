---
trigger: always_on
description: <!-- Loaded on top of global ~/.claude/CLAUDE.md (§1–11). Don't restate global
---

# CLAUDE.md — TerMinal

<!-- Loaded on top of global ~/.claude/CLAUDE.md (§1–11). Don't restate global
     rules here — reference them as "global §N". Keep this lean. -->

TerMinal is a standalone macOS Electron app that hosts Claude Code / Codex CLI
sessions with a software-factory layer on top: tabs for tickets, MRs, scheduled
agents, runs, HITL, docs, and per-session plugin widgets. See
[`README.md`](./README.md) and [`docs/architecture.md`](./docs/architecture.md).

**Status:** Shipped, actively iterating. It is the maintainer's daily driver —
the primary terminal across every other local project.

## TerMinal follows the full PR + human-merge flow (global §8)

TerMinal has graduated from its old direct-to-main exception. It is now a mature
daily-driver, so **agents follow global §8 in full**: work on a feature branch
(prefer a `git worktree` under the shared worktrees root from the global
config, e.g. `<worktrees-root>/TerMinal/<branch>/`),
push the branch, open a PR (`gh pr create`), and **stop for a human merge**.
Never commit or push directly to `main`.

This is enforced: the `block-main-merge.sh` PreToolUse hook IS wired in
`.claude/settings.json` — it blocks `gh pr merge`, `git push … main`, and bare
`git push` while on `main`. (The `TERMINAL_FORCE_MAIN=1` inline override still
exists for human-approved emergency direct-main work, per global §8's FORCE
exception — use it only when explicitly approved.)

**Merge-ready bar** (global §8): code-review verdict `approve` + tests pass +
zero findings at severity ≥ medium. The human runs the merge.

## Release + version discipline (important now that we're PR-first)

Because we no longer release on every commit, the installed
`/Applications/TerMinal.app` can lag `main`. Keep them reconciled:

- **Release from `main`, after a PR merges** — not from feature branches. Easiest:
  Settings → Updates → **Update now** (shown once the app detects it's behind
  `main`) does a full pull + rebuild + reinstall + relaunch with no manual
  commands. `bin/release` builds it itself (runs `bun run dist` as its first
  step) — `bun run release` alone is enough, no separate `bun run dist` needed.
- **Know what's installed:** the build stamp (commit sha + build time) is baked
  in at build (`electron.vite.config.ts`) and shown in **Settings** (top-right).
  Compare it against `git log` on `main` to see if the installed app is current.
  A `-dirty` suffix means it was built from an uncommitted working tree.
- While iterating on a branch, `bun run release` builds *that branch* — the stamp
  will show the branch name; re-release from `main` once merged.

Runbook: [`docs/runbooks/build-and-release.md`](./docs/runbooks/build-and-release.md).

## Commands

```bash
bun install               # at repo root
bun run dev               # dev server with HMR
bun run dist              # build + package (electron-vite + electron-builder)
bun run release           # FULL: pull latest → build → sign → reinstall /Applications/TerMinal.app → relaunch
bun run test              # unit test suite (bun test)
bun run test:ux           # UX suite tier 1 — Playwright drives the real app (needs `bun run build`)
bun run ux:taste          # UX suite tier 2 — AI taste pass, writes a Reports artifact (never a gate)
bunx tsc --noEmit         # typecheck
```

## Architecture pointers

- `src/main/` — Electron main: IPC handlers, agents runtime, schedules, settings.
- `src/renderer/src/tabs/` — one folder per tab. Each tab exports a `Tab` spec
  with `appliesTo` + `Component` + optional `badge`.
- `src/renderer/src/lib/nav.ts` — cross-tab navigation bus
  (`navigateTo(tabId, payload?)`). Used for HITL → Runs, Activity → Tickets, etc.
- The standalone processes are typed modules under `src/`, each BUILT to a
  single self-contained bundle in `bin/` by `bun run build:bin`
  (`scripts/build-bin.ts` holds the table), which `bun run build`/`dist` do for
  you: `src/runner/` → `bin/terminal-cron`, `src/monitor/` →
  `bin/terminal-monitor`, `src/cli/` → `bin/terminal-cli`, `src/mcp/` →
  `bin/terminal-mcp-server`. Edit the TS, never the artifact; the artifacts stay
  committed because host-provision, the agent image and the app all copy them out
  of a checkout, and `src/bin-build-sync.test.ts` fails if any is stale.
  Bundling is also what lets them IMPORT the shared modules
  (`src/runner/state-io.ts` for the crash-safe lock,
  `src/runner/repo-state.ts` for sidecar resolution, `src/shared/monitor-*.ts`
  for the monitor logic) instead of carrying hand-copies — the bundler inlines
  those at build time, so the artifact still resolves nothing at runtime.
- `bin/terminal-cli` — helper exposed inside agent `.sh` bodies for
  ticket/hitl/activity/notify/state subcommands plus MCP passthroughs such as
  `terminal-cli mcp list_agents ...` and
  `terminal-cli mcp request_agent_artifact ...`. Source: `src/cli/`.
- `~/.config/TerMinal/` — runtime state (schedules, cron-runs, agent-state,
  hitl, settings). Use Settings → Open TerMinal config dir to inspect.

## Ticket and agent workflow

Every backlog ticket is owned by exactly one agent via `agent_id`,
`agent_scope` (`repo` | `global`), and `agent_kind` (`classic` | `persistent`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trevormil/TerMinal](https://github.com/trevormil/TerMinal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
