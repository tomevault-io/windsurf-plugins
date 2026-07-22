---
trigger: always_on
description: Orientation for AI agents (Claude, Codex, Cursor) working in this repo. **Read this first.**
---

# AGENTS.md

Orientation for AI agents (Claude, Codex, Cursor) working in this repo. **Read this first.**

## What this is

`cc-multi-cli-plugin` is a Claude Code plugin that **offloads heavy coding work to external CLIs** — Codex, Cursor (headless `agent -p`), Antigravity, and OpenCode (headless `opencode run --format json`) — so the orchestrating Claude session spends as few tokens as possible. The plugin's entire value is *token reduction by delegation*. Keep that goal central to every change.

## The golden rule

Claude does as little thinking as possible; the external CLI does the work.

- The per-command subagents (`plugins/multi/agents/codex-*.md`) are **thin forwarders**: they build exactly one companion command, run it, and return stdout unchanged. They must not read files, reason about the task, or "help."
- Forwarder models are tuned by role. Forwarders that **frame/route** the prompt — `codex-execute`, `codex-rescue`, and the cursor/antigravity ones — run on **Sonnet**, because shaping the task well (model/effort choice, prompt framing) materially improves what the external CLI then produces (this mirrors the official `codex-plugin-cc` rescue subagent). Forwarders that do **no framing** and only bridge to the companion — `codex-review` — run on **Haiku**, the correct cheapest model for a pure forwarder. Keep them all thin either way: never add file reading or task reasoning.
- Do **not** spawn fleets of Claude subagents to do or validate work in this repo — that defeats the plugin's purpose. Validate with `npm test` (offline) or by delegating to a CLI.

## The request chain

```
/codex:<cmd>  →  multi:codex-<role> (forwarder, Haiku)  →  multi-cli-companion.mjs <subcommand> --cli <name>
              →  lib/adapters/<name>.mjs  →  (codex app-server broker | cursor headless `agent -p` | antigravity headless `agy -p` | opencode headless `opencode run --format json`)  →  external CLI
```

See `ARCHITECTURE.md` for the full picture and the job / state / broker model.

## Build & test

No dependencies; uses Node's built-in test runner (Node ≥ 20; repo runs on 25).

- `npm test` — fast, **offline** unit tests. Run this to self-verify any change. No CLI calls, no tokens.
- `npm run test:live` — end-to-end; **spawns real Codex** (costs tokens + time). Run only when touching the live path.

**Definition of done** for a change: `npm test` passes, no `DEP0190` warnings, and `CHANGELOG.md` updated for user-facing changes.

## Map

- `plugins/multi/` — the hub plugin: `agents/` (forwarders), `commands/` (slash cmds), `skills/` (forwarder contracts), `schemas/`, `prompts/`, `hooks/`.
- `plugins/multi/scripts/multi-cli-companion.mjs` — CLI entrypoint; dispatches subcommands (`task`, `review`, `adversarial-review`, `status`, `result`, `cancel`, `setup`).
- `plugins/multi/scripts/lib/adapters/` — one adapter per CLI. Interface in `CONTRACT.md`.
- `plugins/multi/scripts/lib/` — shared runtime: broker lifecycle, app-server, job control, render, git. The ACP client layer lives in `lib/acp/` (`client.mjs` = `runAcpTurn` on the official SDK, `resolve.mjs`, `diagnostics.mjs`); a legacy `lib/acp-client.mjs` predates it and is slated for deletion.
- `plugins/{codex,cursor,antigravity,opencode}/` — per-CLI command slices that forward into `multi`.
- `test/` — `unit/` (offline) + `fixtures/` (sandbox helper). `test:live` reuses `plugins/multi/scripts/test/`.

## Landmines

- **Broker lifecycle**: the Codex app-server broker is a detached per-cwd daemon, reused across tasks. The SessionEnd hook reaps the session's *primary*-cwd broker; brokers for any other cwd self-terminate after an idle window (`CODEX_COMPANION_BROKER_IDLE_MS`, default 600000 ms — see `app-server-broker.mjs` + `lib/broker-lifecycle.mjs` → `shouldIdleShutdown`). Set the env to `0` to disable idle shutdown.
- **State is per-cwd**: jobs and brokers key off the working directory; always pass `--cwd`. Parallel agents should use separate worktrees / cwds to stay isolated.
- **`spark`** (`gpt-5.3-codex-spark`) is rejected on ChatGPT-auth Codex accounts — expect a 400; not a bug.
- **Cursor uses headless `agent -p` by default** (ACP is opt-in via `MULTI_TRANSPORT_CURSOR=acp`): on headless the prompt is delivered on stdin (newline-safe), models pass through as flat `--model` names (default `auto`), roles map to flags (`delegate`→agent+`--force --trust`, `research`/`explore`→`--mode ask --force`), and cancel is the generic process-tree kill. `--until-done` loops `--resume` turns. **Cursor's shell is slow/unreliable on Windows** (host-PATH/WSL, open upstream), so `/cursor:delegate` defers build/test verification to the caller (Claude) — file writes and web/codebase reads are unaffected.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [greenpolo/cc-multi-cli-plugin](https://github.com/greenpolo/cc-multi-cli-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
