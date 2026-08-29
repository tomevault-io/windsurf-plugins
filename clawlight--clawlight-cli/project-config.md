---
trigger: always_on
description: A single Rust binary that monitors [Claude Code](https://claude.ai/code) sessions
---

# clawlight

A single Rust binary that monitors [Claude Code](https://claude.ai/code) sessions
in real time. It ships three faces of the same executable:

- **TUI dashboard** (`clawlight`) — a ratatui table of all sessions with live status.
- **Menu bar / system tray daemon** (`clawlight menubar`) — a color-coded Clawd icon
  reflecting aggregate session health; auto-starts at login.
- **Hook backend** (`clawlight hook`) — invoked by Claude Code's lifecycle hooks to
  record per-session status. Replaces the old bash+jq hook script; no shell/`jq` dep.

Optionally mirrors aggregate state to a Seeed XIAO ESP32-C6 status board over USB serial.

Cross-platform: macOS, Windows, Linux. MIT licensed.

## Build / test / install

```bash
cargo build              # debug build
cargo test               # unit tests + end-to-end tests (tests/) of the built binary
cargo clippy             # lint (CI enforces -D warnings)
cargo fmt                # format (CI enforces --check)
cargo install --path .   # build + install to ~/.cargo/bin/clawlight
clawlight install        # register hooks in ~/.claude/settings.json + login autostart
clawlight uninstall      # reverse install and delete ~/.claude/clawlight
```

`clawlight install` is idempotent and also (re)starts the tray daemon, so re-run it
after `cargo install` to make the running daemon pick up a new binary. On macOS it
rewrites and reloads the LaunchAgent via `launchctl bootout`/`bootstrap`/`kickstart`.

**Always redeploy after code changes.** Whenever you change code in this repo, finish
by updating the app that is already running on this machine so it can be tested live:

```bash
cargo install --path . && clawlight install
```

Never test by launching a second daemon (e.g. `cargo run -- menubar`) — macOS has no
single-instance lock, and two daemons mean two tray icons fighting over the same
serial port. Redeploy the installed one instead.

## Subcommands (src/main.rs)

| Command | Purpose |
|---|---|
| *(none)* | Launch the TUI dashboard (`run_tui`); first launch on an unconfigured machine auto-runs the install (see gotchas) |
| `install` / `uninstall` | Hooks + platform autostart (launchd / registry Run key / XDG autostart) |
| `menubar` | Run the tray daemon in the foreground |
| `led [--port]` | Foreground ESP32 LED mirror (debugging) |
| `update <firmware> [--port]` | Serial-OTA push of new board firmware |
| `hook` *(hidden)* | Claude Code hook backend; reads one event as JSON on stdin |
| `event` *(hidden)* | Normalized-event backend for non-Claude harnesses (opencode); reads one event as JSON on stdin |
| `codex-hook` *(hidden)* | Codex shim: reads one Claude-dialect hook payload on stdin, maps it to a normalized event |
| `copilot-hook <event>` *(hidden)* | Copilot shim: reads one per-event payload on stdin (the event name rides on argv), maps it to a normalized event |
| `name <id> <transcript>` *(hidden)* | Detached auto-namer; titles a session via the `claude` CLI |

## Module map (src/)

- **main.rs** — CLI parsing, TUI setup/teardown (panic hook restores the terminal),
  and all install/uninstall + per-platform autostart logic.
- **hook.rs** — the `hook`, `event`, `codex-hook`, `copilot-hook`, and `name` backends.
  Maps Claude hook events (and the harness-agnostic normalized verbs from `event`) →
  `Status`, does the locked read-modify-write of `state.json` via the shared
  `update_state` helper, spawns the detached auto-namer on first `Stop`. `run_event` is
  the multi-harness ingestion path; `run_codex_hook` and `run_copilot_hook` translate
  Codex's Claude-dialect / Copilot's per-event payloads onto the same verbs in-process —
  see "Multi-harness adapters" below.
- **codex.rs** — everything Codex-specific behind the `codex` adapter: `$CODEX_HOME`
  paths, the position-preserving hooks.json merge (Codex trusts hooks by content hash +
  position — never reorder foreign groups, never write Codex's config.toml), thread
  names from `session_index.jsonl`, exec-vs-interactive from rollout `session_meta`,
  and first-typed-prompt extraction for fallback naming.
- **copilot.rs** — everything Copilot-specific behind the `copilot` adapter:
  `$COPILOT_HOME` paths and the wholly-owned `hooks/clawlight.json` registration file
  (one command group per lifecycle event, the event name baked into argv because
  Copilot payloads don't carry it; ownership proven by the `copilot-hook` command
  string — JSON has no comments for the managed-by marker).
- **state.rs** — `HookState`/`SessionStatus`/`Status` types (incl. the optional
  `harness` tag: absent = Claude, `"opencode"` = opencode), `state.json` read/write
  (atomic temp-file + rename), the shared `.state.lock` (`acquire_state_lock`), the
  `reap_ended_sessions` downgrade to `Done` (dead-process reap via `terminal::is_alive`,
  plus the 24h staleness backstop), and `Aggregate` (Red/Yellow/Green/None) health rollup.
- **session.rs** — discovers `~/.claude/projects/*/sessions-index.json`, merges those
  entries with hook state into `DisplaySession`s, resolves display names and sort order.
- **config.rs** — `~/.claude/clawlight/config.json` (LED opt-in, optional `led_port`,
  and `yellow_mode` — how idle sessions color the aggregate).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clawlight/clawlight-cli](https://github.com/clawlight/clawlight-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
