---
trigger: always_on
description: A Go daemon and command-line tool (`cmd/agentbox`, `internal/`) and an Electron desktop app (`desktop/`). Go and Node come from mise (`mise.toml`).
---

# AgentBox

A Go daemon and command-line tool (`cmd/agentbox`, `internal/`) and an Electron desktop app (`desktop/`). Go and Node come from mise (`mise.toml`).

## What it is

AgentBox runs several AI coding agents against one project at the same time, each in a Linux
container of its own so they can't collide.

- **The daemon owns the state.** `internal/daemon` is the control plane: every agent operation goes
  through it, the slow ones as jobs, and it serves the HTTP API from `internal/api` over a unix
  socket at `~/.local/share/agentbox/run/agentbox.sock`. All of the state is one SQLite database,
  `state.db`, in `internal/state`. The CLI is a client of that socket, and so is the app
  (D15).
- **The desktop app is a thin client.** Its main process only relays the socket to the renderer: no
  AgentBox logic, no Incus, no git (D19). The API types the renderer uses are
  **generated from Go** into [`desktop/src/shared/api.ts`](desktop/src/shared/api.ts); change
  `internal/api` and `TestTypeScriptTypesAreUpToDate` fails until you run
  `UPDATE_TS=1 go test ./internal/api` (D18).
- **An agent is a machine, a worktree and a branch.** `agentbox create` gives it an Incus container,
  a git worktree on `agentbox/<slug>`, a branch named after its work — the slug given to create, or one made from its title or task (`agentbox/` is the project's branch prefix, and can be changed; `internal/agent/branch.go`) — its own network and a tmux terminal with its AI tool already
  running. A project also has a **lead** —
  its chat — which runs on the host with no machine of its own, so a project you have never chatted
  with costs nothing; it directs the project's agents over MCP and never does the work itself.
- **Three AI tools:** Claude Code, Codex and OpenCode. The app's chat drives all three through
  **ACP** adapters (`internal/acp`, `ChatAdapters` in `internal/agent/chat.go`) — Claude Code and
  Codex through adapters of their own, OpenCode through its own `acp` subcommand
  (D40).
- **Project memory is what the project knows, kept across agents.** Events, memories, working
  memory, tasks, artifacts and reports, as tables in the same database and searched with FTS5 — the
  store calls no model and embeds nothing. Around it: automatic capture from the daemon's own
  chokepoints, consolidation of events into memories (by default on the project's tool's cheap
  model, in a session of its own), and a context builder that gives each agent the slice of memory
  its task needs. It is in `internal/memory` (D72).
- **What chats spend is kept in a token ledger.** `token_usage` gets one row per model per turn,
  written by the chat as each turn ends (`internal/chat/tokens.go`) from what the ACP adapters
  report. It's read through `agentbox tokens` and each project's Tokens tab. Every Claude Code chat
  compacts at the installation's compact window, 200k tokens by default, because each model call
  resends the whole conversation. In Claude Code the desktop tools belong to a `desktop` subagent,
  so screenshots don't stay in an agent's context. Searches go to an `Explore` subagent on Haiku,
  and at most three subagents run at once. Each Claude account's five-hour and weekly limits are
  kept as its chats report them. Subagents show in the chat as cards
  (D83–D86).
- **Project notes** are one markdown file per project (`internal/notes`), written by the user in the
  app and added to by the lead under `## From the lead`. They are folded into every agent's brief,
  the instructions each AI tool gets about its machine, rendered by `internal/brief` from
  [`brief.md.tmpl`](internal/brief/brief.md.tmpl) — which is also where the memory section lands.
  The brief has golden tests: change the template and run `go test ./internal/brief -update`.

## On a Mac

AgentBox runs in a Linux VM there, made with Lima, and nothing of the above is ported: the daemon,
Incus and the agents are the Linux ones, inside the VM. The macOS `agentbox` is a front end
(`internal/hostvm`): `agentbox vm …` makes and manages the VM, and every other command runs in it
through `limactl shell`. The daemon's socket is forwarded to the Mac at its usual path, so the app
is the same client it is on Linux; the Mac's home is shared at the same path, and the agents'
worktrees go there (`AGENTBOX_WORKTREES`). Android is off in the VM. The whole design is
D92; on Linux,
`AGENTBOX_FRONT_END=vm AGENTBOX_VM_TYPE=qemu` runs the front end against a QEMU VM, to test it
without a Mac.

## Conventions

- **Migrations are appended, never edited.** `migrations` in
  [`internal/state/state.go`](internal/state/state.go) is one ordered list, tracked with
  `PRAGMA user_version`. Editing a past entry leaves already-migrated databases behind.
- **Record a decision** when the shape of something is worth recording — what the context was, what
  was decided, what was rejected, and what proves it — in the pull request that makes it. The code
  cites earlier decisions by number (D1–D92); their records aren't in this repository.
- **Explain a feature worth explaining** in its pull request: what it does, what was checked, where
  the code is, and what it still can't do.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leciric/agentbox](https://github.com/leciric/agentbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->
