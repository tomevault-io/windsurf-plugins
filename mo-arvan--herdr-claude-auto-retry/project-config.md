---
trigger: always_on
description: A [herdr](https://herdr.dev) plugin that auto-resumes Claude Code panes after an Anthropic rate limit or a transient server error. It is a herdr-native reimplementation of the unmaintained, tmux-based [`cheapestinference/claude-auto-retry`](https://github.com/cheapestinference/claude-auto-retry). herdr already multiplexes terminals and exposes agent detection + pane read/send over its CLI, so we talk to herdr directly instead of nesting tmux. `README.md` is the user-facing doc; this file is the 
---

# AGENTS.md - herdr-claude-auto-retry

## Overview

A [herdr](https://herdr.dev) plugin that auto-resumes Claude Code panes after an Anthropic rate limit or a transient server error. It is a herdr-native reimplementation of the unmaintained, tmux-based [`cheapestinference/claude-auto-retry`](https://github.com/cheapestinference/claude-auto-retry). herdr already multiplexes terminals and exposes agent detection + pane read/send over its CLI, so we talk to herdr directly instead of nesting tmux. `README.md` is the user-facing doc; this file is the agent operating manual.

## Commands

- `npm test` - Node's built-in runner, zero dependencies.
- `npm run preflight` (every release gate, read-only: git state, changelog, suite under `npm run coverage` floors, nothing skipped so the herdr and Claude contract tests ran, review checklist) and `npm run release -- X.Y.Z` (bump, promote the CHANGELOG, commit, tag - never push). Process and the judgment calls it cannot make: `CONTRIBUTING.md`.
- `npm run scan` - leak scan over tracked files; also runs in `npm test` and in `.githooks/pre-push` (enable with `git config core.hooksPath .githooks`).
- `herdr plugin link .` - load the working tree into a running herdr for live testing.
- `herdr plugin action invoke claude-auto-retry.<watch-all|arm|status|stop|logs>` - drive the actions.
- Local E2E without a herdr server: point `HERDR_BIN_PATH` at `test/fixtures/fake-herdr.js` (serves canned `pane list/get/read` JSON and records `send-*`; give it a `screen` and it becomes a small reactive Claude TUI model - vim mode, menu, input line, submit - so whole recoveries run end to end); drive the monitor with `node bin/main.js monitor <terminalId> <paneId>`.

## Layout

- `herdr-plugin.toml` - manifest: one `[[startup]]` hook (-> `watch-all`, D12), two event hooks (`pane.agent_detected` + `pane.agent_status_changed`, both -> `hook-agent-detected`; D12) + five actions, all run via `["/bin/sh", "launch.sh", "<subcommand>"]`. `min_herdr_version` is 0.7.5, the release that added `[[startup]]`. `test/release.test.js` keeps the manifest, the `HANDLERS` table, the two version fields, the CHANGELOG and the documented herdr floor in step.
- `launch.sh` - POSIX node resolver (fnm/nvm/mise/asdf/volta/`$HERDR_NODE`); see D5. Runs `bin/main.js <subcommand>`.
- `bin/main.js` - the single entrypoint: dispatches the hook, the actions, and the long-lived `monitor` subcommand (D6). Thin glue; logic lives in `src/`.
- `src/monitor-core.js` - the transport-agnostic rate-limit state machine (the unit-tested core).
- `src/herdr.js` - the herdr CLI adapter.
- `src/registry.js` - atomic per-`terminal_id` monitor lock (exactly one monitor per pane).
- `src/{patterns,time-parser}.js` - detection and reset-time math.
- **This repo is public and its fixtures are screenshots of real sessions.** Never paste a live pane capture in verbatim: keep the shape, replace the content. `scripts/scan-private.mjs` is the guard (generic patterns shipped; maintainer names in a gitignored `.private-markers`). Gitignored files that matter - `PROGRESS.md`, `.private-markers` - are symlinks into a private repo, so they are backed up without ever entering this history.
- `scripts/{release,changelog}.mjs` - the local half of the release; `.github/workflows/release.yml` is the remote half, triggered by the tag push. **A plain `herdr plugin install` takes the default branch, so pushing `main` is the release** and the only rollback is a revert. `plugin install --ref <branch|tag>` fetches a specific ref instead, which is what makes `next` a staging channel and lets a user pin a tag; there is still no `plugin update`, so refreshing means reinstalling.

## Conventions

- ESM (`"type": "module"`), Node `>= 18`, no runtime dependencies (built-ins only).
- All herdr access goes through `src/herdr.js`; never shell out to `herdr` elsewhere. `test/herdr-contract.test.js` runs that exact CLI surface against the real `herdr` when it is on PATH (skipped otherwise), so a renamed flag fails locally instead of in a release.
- Behavioral logic goes in `src/` (pure, unit-tested with a fake adapter); `bin/main.js` only wires I/O. It is covered by `test/e2e-monitor.test.js`, which spawns the real entrypoint against `fake-herdr`.

## Gotchas

- Plugin event hooks can only subscribe to herdr's `PLUGIN_HOOK_EVENT_KINDS`; `pane.output_changed` is deliberately excluded (too high-volume). So activation is `pane.agent_detected` + `pane.agent_status_changed` (D12) + a polling monitor, not output-change events.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mo-arvan/herdr-claude-auto-retry](https://github.com/mo-arvan/herdr-claude-auto-retry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
