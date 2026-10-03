---
trigger: always_on
description: manages the AudioContext; `unlockAudio()` on canvas mousedown resumes it (webviews start
---

# Agent Campus — Compressed Reference

Electron desktop app: a pixel-art campus where AI agents (Claude Code sessions) are animated
characters. Every project is an office; agents are characters you watch work and dispatch.

Single target. This was once a dual-target repo that also built a VS Code extension out of
`src/`; that target was deleted in `55ede0a` for falling ~2 months behind. What survives is
`src/core/` (host-agnostic backend) and `shared/protocol.ts` (the message contract) — still
split that way because the split is good structure, not because a second host is coming. The
upstream project it was forked from still ships an extension.

## Architecture

```
shared/
  protocol.ts                 — THE message protocol: HostToWebviewMessage / WebviewToHostMessage
                                discriminated unions + shared data shapes (SpriteData, FurnitureAsset,
                                AgentSeatMeta). Types-only; imported by src/, electron/, webview-ui/.

src/core/                     — Host-agnostic backend core (no vscode/electron imports)
  types.ts                    — CoreAgentState, TrackerContext (agents+watchers+timers+send()), Send
  constants.ts                — Shared timing/truncation/PNG/layout constants
  transcriptParser.ts         — JSONL parsing: tool_use/tool_result → send() messages; idle detection;
                                per-turn TurnStats tally → agentSuggestions on turn_duration
  actionSuggestions.ts        — End-of-turn heuristics: TurnStats (edits/errors/ranTests) →
                                AgentActionSuggestion[] (Review/Test/Commit/Investigate buttons)
  roles.ts                    — Dispatch roles: charter (~150-token systemPrompt append) + tool policy
                                (SDK disallowedTools; qa/security are read-only) per role id; ids match
                                ROLE_SKIN_DEFS. roleForTaskText ("[qa] ..." tag → role) and
                                roleForSubagentType (Task subagent_type keyword → skin id). NO bundled
                                knowledge by design — project knowledge stays in workspace
                                CLAUDE.md/.claude/skills and loads on demand
  timerManager.ts             — Waiting/permission timer logic
  fileWatcher.ts              — startFileWatching/stopFileWatching/readNewLines (fs.watch + watchFile +
                                poll, byte-level UTF-8-safe line carry, truncation reset, error handlers)
  assetLoader.ts              — PNG parsing, sprite conversion, asset/layout loading, sendX(send, ...) helpers
  layoutPersistence.ts        — Just isValidLayout() now. It used to own the VS Code host's single
                                ~/.pixel-agents/layout.json (atomic write + cross-window watcher);
                                the campus replaced that with one file per workspace, owned by
                                electron/main.ts, so the rest was deleted

electron/                     — Electron desktop host (imports src/core; tsconfig rootDir=.. →
                                dist-electron/electron/main.js + dist-electron/src/core/)
  main.ts                     — Main process: window, node-pty terminals (main constructs ALL commands;
                                renderer only sends keystrokes). INTERNAL SESSIONS ONLY: agents exist solely
                                for sessions the app spawned (no scanning of external sessions). TWO AGENT
                                KINDS: 'terminal' (PTY running claude, xterm tab) and 'chat' (Agent SDK
                                session, rich chat tab — the default for "+ Agent"). Both kinds register a
                                transcript watcher via a caller-supplied sessionId, so office characters
                                animate identically for both. /clear detection scans each agent's project
                                dir; a new JSONL is reassigned to the agent there whose PTY/composer most
                                recently received input. Scrollback replay (pty-ready→pty-replay),
                                seat/palette persistence keyed by SESSION id, settings, folder picker on
                                agent creation, login-shell PATH fix, sandbox+navigation guards.
                                Per-workspace layout files (~/.pixel-agents/layouts/) are sent on
                                webviewReady AND hot-reloaded via a layouts-dir watcher (debounced,
                                own-write suppression) — external edits apply without restart
  workspaces.ts / todos.ts / usage.ts — persisted registries (~/.pixel-agents/): offices, per-workspace
                                task lists, per-turn cost ledger (per-turn DELTAS derived from the
                                SDK's cumulative total_cost_usd — summing that field raw inflates
                                spend quadratically; figures are API-list estimates, not plan spend)
  openAgents.ts               — Which agents were open, so a restart can offer them back
                                (~/.pixel-agents/open-agents.json). Written on every open and close,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mateovalle/agent-campus](https://github.com/mateovalle/agent-campus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
