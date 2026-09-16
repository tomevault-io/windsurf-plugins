---
trigger: always_on
description: The rules that are easy to break without noticing, and the file that owns each
---

# AGENTS.md

The rules that are easy to break without noticing, and the file that owns each
one. Stack and build commands: [README.md](README.md). Isolated dev window and
MCP bridge, and the end-to-end scenarios in `e2e/`:
[docs/development.md](docs/development.md). Tags and signing:
[docs/releasing.md](docs/releasing.md). Reading what any host logged, and the
`logs` tool that answers "what happened to thread X":
[docs/logging.md](docs/logging.md).

Everything here is a good default rather than a law, and the person asking
outranks it. The four rules below are the exception: break one and something
ships broken without a check failing.

## Words

- **you**: the agent reading this and changing boite. Most changes to boite are
  made from inside boite, so you are probably running as one of its threads, in a
  detached worktree of the repo you are editing.
- **user**: the person driving those threads. **agent** or **provider**: one of
  the ten CLIs boite launches (claude, codex, opencode, cursor, antigravity,
  copilot, grok, hermes, pi, muse).
- **project**: a folder boite knows about. **thread**: one terminal in it, with
  its own PTY, its own worktree and at most one agent conversation bound to it.
- **group**: the set of panes drawn for one thread. Every group is mounted at
  once and all but one are `visibility: hidden`.
- **boite**: this app. **a boite**: one running instance, which may be the
  desktop window or a `boite-server` other clients connect to.

## The three ways to hurt yourself

1. **Killing by pattern.** Never `pkill -f`, `taskkill /IM`, or kill a pid you
   found by matching a name, a path or a worktree string. Your own process, the
   user's other agent threads and the app drawing them all carry "boite" and
   often this worktree's path in their argv. Kill only a pid you captured at
   spawn.
2. **Touching the live install.** The user's real database, scrollback and window
   state sit in `com.boite.legacy`'s app data directory and are open while you
   work. Copying out of it is fine and is the only realistic test data. Never
   open it read-write, never point a server at it, never tidy it. That is what
   `bun run dev:isolated` is for: a separate window on port 1430 under
   `dev.boite.dev`, with its own database. A plain `tauri dev` refuses to start
   anyway, the release instance holding the single-instance lock.
3. **Deleting a worktree by hand.** A thread's worktree holds a junction onto the
   project's session store, and on Windows a delete that meets a junction walks
   into it, so an `rm -rf` on a worktree can take every transcript in the project
   with it. Go through `worktree.remove`, which unlinks first.

## Four rules with no exceptions

1. **No literal user-facing string in a `.svelte` file**, `title`, `placeholder`
   and `aria-label` included. `t()`, with the key a literal at the call site: a
   template literal defeats the `MessageKey` derivation and the French
   `Record<MessageKey, string>` at once, so it ships an untranslated string
   `bun run check` cannot see. A key that has to vary goes on the data instead.
2. **No `invoke` in a component.** Everything goes through `backend()`, which is
   Tauri locally and a WebSocket when the boite is a server. Inside the Tauri
   backend, import `invoke` from `backend/tauri/ipc.ts`, never from
   `@tauri-apps/api/core`: that door writes a `warn` when a command refuses,
   then re-throws untouched.
3. **No SQL in the webview.** Projects, threads, todos and settings go through
   `commands::records`, and `db.ts` holds no statements. A row rule belongs in
   `boite_core::store` or `command::records`, where both hosts get it; written on
   one side only it drifts, and the drift rewrites rows the other host wrote.
4. **No `$state` writer that reads the state it writes.** It subscribes its
   caller's `$effect` to its own output. Cost so far: `note()` skipped a
   same-value write, the frame's `load` re-triggered the effect arming the stall
   timer, and every browser pane reported `loading` forever.

## One bus, two front doors

A capability is a value on `boite_core::command`; the Tauri commands and the
server are codecs over it. Adding one is a variant, its name in that domain's
`ALL_METHODS`, the arms the compiler then demands, and a `#[tauri::command]`
calling `on_bus`. The server needs nothing, `command::handles` routes it.

`Command::decode` routes on each domain's own method list, not on the wire
prefix, so `project.inspect` (files, a folder that is not a project yet) and
`project.list` (records) coexist. A test asserts no two domains claim one name:
a duplicate decodes into the wrong domain with the wrong envelope rather than
failing.

## A thread's look is derived, never stored

The model tint, the icon and the fastpick combo are computed at render from
`cmd` and `args` (`fastpick/threadAccent.ts`, `parseCombo`). Stored on the row
they would apply to new threads only, and would miss a thread that renamed
itself. `withoutBarColor` drops the `/color` argument old rows still carry as the
argv goes past, rather than rewriting the row: a thread's command line is the
user's to read and edit.

A process can rename itself. `ESC ] 1337 ; boite ; launch = {json} BEL` on its

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beboite/boite-legacy](https://github.com/beboite/boite-legacy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
