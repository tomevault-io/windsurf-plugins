---
trigger: always_on
description: Shared maintainer guidance for coding agents working in the Luvus repository.
---

# AGENTS.md

Shared maintainer guidance for coding agents working in the Luvus repository.
This file is the repository-level source of truth. Tool-specific instruction
files may add guidance, but they must not contradict it.

## Start with current evidence

Luvus changes quickly. Before describing behavior or editing a subsystem:

1. Check `git status --short --branch` and preserve every unrelated change.
2. Inspect the current implementation and its focused tests.
3. Use `cargo run --quiet -- help all` or `src/cli.rs` for the installed command
   surface.
4. Use `website/src/content/docs/` for public product documentation.
5. Treat ignored files under `docs/` as plans and historical handoffs, not as
   proof that a feature exists.

When remote state matters, compare the checkout with `origin/main`. Do not
silently change branches, fetch, rebase, or rewrite the user's work merely to
make the checkout match upstream.

## Product model

Luvus is mission control for AI coding agents. It is a single Rust binary with
several roles:

- With no command, it starts or attaches a thin TUI client.
- A detached server owns workspaces, tabs, panes, PTYs, terminal grids, agent
  state, persistence, modules, orchestration, API dispatch, and rendering.
- CLI commands send bounded JSON requests to the selected server and exit.
- Named sessions are independent server namespaces selected with
  `--session <name>`, `session attach <name>`, or the on-demand desktop/mobile
  session switcher. Switching starts the target if needed without detaching
  other clients.
- `--local` is a monolithic development escape hatch.
- `--remote <host>` attaches directly through an SSH byte bridge. Saved machine
  profiles place remote workspaces beside Local in one TUI; each profile selects
  one remote named session. Remote endpoint connections and viewport state
  belong to the client, while the remote server retains its own PTYs.
- `web` starts an optional foreground browser bridge for the selected session.
  It is loopback-only and read-only by default; `web --control` opts into
  terminal and workspace control. Normal TUI/server startup does not open an
  HTTP port. Stopping the bridge revokes its authority without stopping the
  server, PTYs, or other clients.
- `uhp access` starts a temporary loopback NDJSON gateway for independent
  transport providers. It is read-only by default; control is explicit,
  scoped, and paired. Finite authority advertises `authority.expires_at`;
  `--no-expiry` is process-bound and advertises
  `authority.expires_on_close:true`. Shutdown revokes authority in either mode.

There are two local endpoints per server:

- the newline-delimited JSON control API;
- the binary client transport for input, full frames, frame diffs, clipboard,
  notifications, sound, detach, and shutdown messages.

Clients are disposable. The server is the single writer of application state.
Detaching a client leaves panes alive. Stopping the server ends its live PTYs;
the next server restores layout and resumes supported native agent sessions
where possible.

## Non-negotiable development safety

- Work in the current project directory. Do not create another worktree, clone,
  or development directory under `/tmp`, `/private/tmp`, or elsewhere unless
  the user explicitly requests it.
- Never stop, restart, attach to, benchmark, or delete the user's production
  Luvus server during development.
- Debug builds normally use `~/.luvus-dev/`; installed release builds use
  `~/.luvus/`. An inherited `LUVUS_SOCKET_PATH` or session selector can still
  route a debug command to another running server.
- Before a debug lifecycle or integration test, run outside a managed Luvus
  pane or explicitly isolate the home and remove inherited socket/session
  selectors. On Unix, a safe pattern is:

  ```bash
  env -u LUVUS_SOCKET_PATH -u LUVUS_SESSION \
      LUVUS_HOME="$HOME/.luvus-dev" \
      ./target/debug/luvus --session agent-test server restart
  ```

- Use the exact binary under test. Confirm the executable, selected home,
  session, and socket before interpreting runtime results.
- Never use destructive Git commands, discard user changes, or stage unrelated
  files.
- Do not commit, amend, push, tag, open a PR, merge, publish, or edit releases
  unless the user explicitly authorizes that action.
- Do not add AI co-author trailers unless the user explicitly requests one for
  that commit.
- After implementation, report changed files, verification performed, remaining
  caveats, and a copyable Conventional Commit message.

## Architecture and code map

`src/main.rs` performs process-role and session routing. The important ownership
boundaries are:

- `src/app/`: `App`, input, dispatch, settings, dashboards, workers, and domain
  state. Application mutations converge on the server/local event-loop thread.
- `src/layout.rs`: pure binary space-partition pane geometry. Layout leaves store
  IDs; live panes and native views remain flat in `App`.
- `src/terminal/pty.rs`: portable PTY creation, reader/writer/reaper lifecycle,
  environment propagation, and cancellation. `src/terminal/host_input.rs`
  narrowly reconstructs Windows console bracketed paste before dispatch.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RizRiyz/luvus](https://github.com/RizRiyz/luvus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
