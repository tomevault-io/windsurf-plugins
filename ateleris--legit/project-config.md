---
trigger: always_on
description: LeGit is a desktop **git GUI**: a **Tauri 2.x** app with a **Rust** backend and a
---

# LeGit — project guide for Claude

LeGit is a desktop **git GUI**: a **Tauri 2.x** app with a **Rust** backend and a
**React + TypeScript** frontend. The canonical spec lives in `design/DESIGN-v0.*.md`
(plus dated design notes in `design/`); code comments reference its `§` sections.

## Layout

- `crates/legit-core/` — pure git logic, UI-agnostic.
  - `runner.rs` — `GitRunner`, the single chokepoint that invokes `git`.
  - `backend.rs` — `GitBackend` trait; `cli_impl/` — `GitCliBackend` + `parsers/`.
  - `types.rs` — domain types crossing the IPC boundary (serde + specta).
- `src-tauri/` — the Tauri app: `commands/` (IPC commands), `state.rs`
  (`AppState`, per-repo `RepoSession`), `watcher.rs` (filesystem watcher).
- `src/` — React frontend: `panels/` (UI), `store/` (zustand), `lib/`
  (command wrappers + types), `theme/`, `icons/`, `styles/`.
- Remote-repo crates: `crates/legit-watch` (extracted watcher core),
  `crates/legit-proto` (agent wire protocol), `crates/legit-host` (`Host`
  trait: Local + Remote impls), `crates/legit-agent` (the binary deployed
  into a WSL distro); `src-tauri/src/remote/` (locator, wsl.exe transport,
  connection lifecycle).

## Architecture & key decisions

**Git is run via the CLI, not a library.** Every git invocation goes through
`GitRunner` (hardened env: `GIT_EDITOR=false`, `GIT_TERMINAL_PROMPT=0`,
`LANG/LC_ALL=C.UTF-8`; cancellable via `OperationId`). `run` / `run_with_op` /
`run_with_stdin` / `run_with_env` / `stream`. Parsers are **pure** `text -> type`
functions in `cli_impl/parsers/`, and each command's format string is a constant
next to its parser so the contract lives in one place.

**Env vars beat `-c` config - relax hardening via `run_with_env`.** Git gives
environment variables (`GIT_EDITOR`, ...) precedence over ALL config, so a
`-c core.editor=...` can never undo the runner's `GIT_EDITOR=false` (this
silently broke `merge --continue` / `rebase --continue` until the integration
harness caught it). When one command must relax a hardening default, pass a
per-invocation override through `run_with_env` (applied after the base env, so
it wins) - the continue/skip commands run with `GIT_EDITOR=true` to accept the
prepared message unchanged.

**Remote repositories run through a Host seam (WSL v1).** A repo lives on a
`Host` (`legit-host`): every repo-side action — git spawn (`GitExecutor`,
which now also carries `stream`/`cancel`), file access (`RepoFs` +
`HostPath`, never raw `std::fs` on repo paths), the watcher, helper spawns —
goes through the session's host, so a WSL repo behaves like a local one. The
cut is at the EXECUTOR level: the `legit-agent` deployed into the distro is a
dumb git-runner + fs + watcher host over an NDJSON stdio protocol
(`legit-proto`, bidirectional — credentials relay agent→app), and ALL
parsers/flows stay in `legit-core`, so `remote_git_flows.rs` runs the entire
real-git suite through a spawned agent (the gate — keep it green). Repos are
identified by LOCATOR strings (`wsl://<distro>/<path>`; local = bare path,
persisted bookkeeping unchanged), hashed by `repo_hash_locator` (local hashes
pinned byte-identical by test). Reconnect swaps the connection inside
`RemoteHost` (`HostConn`) so sessions/ids survive `wsl --shutdown`. See
`design/2026-08-31-remote-repositories-wsl.md`.

**Backend logic is testable without git (executor seam).** `GitCliBackend` is
generic over the `GitExecutor` trait (`executor.rs`; default `GitRunner`, so
production code never names it). Composed flows are tested at two levels, and a
new composed flow or output-classification assumption needs both:
`cli_impl/flow_tests.rs` scripts a `FakeExecutor` that asserts the exact git
command sequence (incl. what must NOT run, e.g. no `stash pop` after a
clean-tree auto-stash); `crates/legit-core/tests/git_flows.rs` validates the
encoded assumptions against the real binary in tempdir repos (pins local
config: identity, no signing, no autocrlf). Both run in
`cargo test -p legit-core`.

**Commands & bindings.** Backend commands are `#[tauri::command] #[specta::specta]`,
registered in `src-tauri/src/lib.rs` (`collect_commands!`). specta regenerates
`src/lib/bindings.ts` **when the app runs** (debug), not at `cargo build`. The
frontend actually calls **hand-written wrappers** in `src/lib/commands.ts`
(`invoke(...)`), with types **hand-mirrored** in `src/lib/types.ts` (bindings.ts
is the generated reference). Add new commands in both places.

**Panels are dockview-based.** `src/panels/registry.tsx` declares every panel
(`PanelDescriptor`: id, scope global/repo, `summons`, `defaultPlacement`).
Panels open/focus each other through the **summon** mechanism
(`src/store/summon.ts`): `summon(id, payload)` opens-or-focuses and delivers a
payload (queued until mount); `notifyIfOpen(id, payload)` updates a panel only if
it's already mounted (never opens it). `swapSummon` shares one slot between two
panels.

**Theme system (palette → token → CSS var).** A theme has a **palette** (named
colours; values may include alpha hex like `#4a9eff33`) and **tokens** (each
token maps to a palette entry). `applyTheme` writes CSS custom properties
(`--token-name: var(--palette-entry)`); `resolveTheme` merges over

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ateleris/LeGit](https://github.com/ateleris/LeGit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
