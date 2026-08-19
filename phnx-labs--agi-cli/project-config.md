---
trigger: always_on
description: A monorepo housing the `agents` CLI and AGI EXT, the VS Code extension, plus their
---

# agents-cli (monorepo)

A monorepo housing the `agents` CLI and AGI EXT, the VS Code extension, plus their
shared libraries and native helpers. Install, configure, run, and dispatch AI
coding agents (Claude, Codex, Gemini, Cursor, OpenCode, OpenClaw, Grok, Droid, …)
from one place.

> Phoenix Labs OSS · Apache-2.0.

**Two main projects live here:** (a) the **agents CLI** — [`apps/cli`](apps/cli),
the published `@phnx-labs/agents-cli` — and (b) the **CLI's VS Code extension,
AGI EXT** — [`apps/ext`](apps/ext). Everything else (`apps/ios`,
`native/computer-*`, `packages/*`) is a **helper app / library for one feature**,
not a main project.

**This file is the repo map + repo-wide policy — it deliberately stays shallow.**
**Read the nearest component `AGENTS.md` (recursively) before working in it** — for
Claude that's `CLAUDE.md`, a symlink to the same file — and keep going down: the
component file, not this one, is where component-specific detail lives. Every
component with a real `AGENTS.md` carries `CLAUDE.md`/`GEMINI.md` symlinks to it.

## Purpose — keep agents running, land work end to end

agents-cli is a power user's control plane for running many coding agents at once and
driving each one to a **landed** result (merged, shipped, verified), not just started.
Starting an agent is the easy part. The hard part is that agents stall: they stop
mid-task, ask a question and idle, make a statement ("I won't continue…") and sit, fail
to reach for the browser or a secret they already have, or hand work back instead of
finishing it. Every reliability surface in this repo — the daemon **watchdog**,
`needs-you` detection, **resume/restore**, session-status truth, and the AGI EXT
**Fleet** panel — exists for one job: notice an agent that has stopped making progress
and get it moving again, so work lands end to end without a human babysitting every
session.

**Design consequence — rank by progress, not by liveness.** A *running* session is
making progress and needs nothing from the operator. The sessions that need a human are
the ones that have **stopped** progressing: blocked on a real prompt, stalled on a
statement, **idle mid-task**, or crashed. Idle-but-unfinished work is the
**highest-risk** state, not the lowest, because it is the most likely to be silently
abandoned with no progress ever made. So any status or attention surface (the Fleet
panel, `sessions`, notifications) surfaces not-progressing work **first** and collapses
the healthy running set; it never buries idle work below running work. `done` is a
distinct terminal state from `idle`: an idle session that is genuinely finished is safe
to fold away, while an idle session that is unfinished is exactly the one to raise.

## Repo map

```
apps/
  cli/        @phnx-labs/agents-cli — the `agents`/`ag` CLI (the published npm package)
  ext/        AGI EXT — the VS Code extension + its React UI + Electron app (publisher: swarmify, swarm-ext)
  ios/        Fleet Cockpit — iOS/iPadOS control-plane app (AnchorKit SwiftPM lib + Cockpit SwiftUI); steers the fleet, never a compute worker
native/
  computer-mac/   Swift daemon behind `agents computer` (Accessibility + screen capture)
  computer-win/   C#/.NET daemon behind `agents computer` on Windows (UI Automation)
packages/
  session-tracker/  @agents/session-tracker — SessionStart hook that WRITES live-session state
  agi-cli/          @phnx-labs/agi-cli — DEPRECATED alias; re-exports the canonical @phnx-labs/agents-cli
  swarmify-mirror/  legacy npm-redirect stub (@companion/agents-cli → @phnx-labs/agents-cli)
assets/ demo/ website/   Brand, launch demo, landing (repo-root, not shipped in any tarball)
```

| Component | What it is | Read |
|---|---|---|
| [`apps/cli`](apps/cli) | The CLI — version mgmt, config sync, sessions, teams, cloud, browser, computer, secrets | [AGENTS.md](apps/cli/AGENTS.md) · [README.md](apps/cli/README.md) |
| [`apps/ext`](apps/ext) | AGI EXT VS Code extension — spawns agent terminals as tabs, Fleet dashboard, dispatch | [AGENTS.md](apps/ext/AGENTS.md) · [README.md](apps/ext/README.md) |
| [`apps/ios`](apps/ios) | Fleet Cockpit — iOS/iPadOS control-plane app over the anchor (`agents serve --control`) | [AGENTS.md](apps/ios/AGENTS.md) · [README.md](apps/ios/README.md) |
| [`native/computer-mac`](native/computer-mac) | macOS `agents computer` backend (Swift) | [AGENTS.md](native/computer-mac/AGENTS.md) · [README.md](native/computer-mac/README.md) |
| [`native/computer-win`](native/computer-win) | Windows `agents computer` backend (C#/.NET) | [AGENTS.md](native/computer-win/AGENTS.md) · [README.md](native/computer-win/README.md) |
| [`packages/session-tracker`](packages/session-tracker) | Live-session **writer** (SessionStart hook) | [AGENTS.md](packages/session-tracker/AGENTS.md) · [README.md](packages/session-tracker/README.md) |
| [`packages/agi-cli`](packages/agi-cli) | Deprecated alias — re-exports the canonical CLI | [README.md](packages/agi-cli/README.md) |
| [`packages/swarmify-mirror`](packages/swarmify-mirror) | Deprecated npm-redirect stub | [README.md](packages/swarmify-mirror/README.md) |

**No JS workspaces.** Each package self-installs (`bun install` inside it). There is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phnx-labs/agi-cli](https://github.com/phnx-labs/agi-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
