---
trigger: always_on
description: > Coding charter for AI agents working in this repo. **Rules and constraints only.**
---

# AGENTS.md — java-agent-debugger (`jdbg`)

> Coding charter for AI agents working in this repo. **Rules and constraints only.**
> For architecture, module layout, command/tool surface, output schema, dependencies, and status,
> read **`DESIGN.md`** (the factual design-and-implementation reference). This file does NOT duplicate it.

## What this project is (one paragraph)

`jdbg` is a cross-platform **Rust CLI** (binary `jdbg`, crate `java-agent-debugger`, edition 2024) that lets an
AI agent **debug Java interactively** through the JDK's `jdb` compatibility backend plus an optional Java JDI
sidecar backend — prompt-aware (never sleep-based), stateful (a background daemon keeps sessions alive across
calls), Windows-first. It is consumed two ways: the **CLI** and an **MCP server** (`jdbg __mcp`, native tool calls
for Claude Code, Codex, and OpenCode). Full detail in `DESIGN.md`.

## Binding constraints (do not violate without asking)

These are settled decisions. Changing them needs explicit user sign-off.

- **Keep blocking boundaries where they are settled.** The existing `jdb` engine, daemon registry, daemon IPC,
  and per-session command locking remain `std::thread` + channels + blocking IO unless explicitly redesigned.
  `tokio` is allowed for `rmcp` MCP serving and the JDI sidecar transport/lifecycle subsystem.
- **No temp files, no shell, no sleeps.** Commands are written straight to `jdb`'s stdin. Readiness is detected
  by reading until the prompt, never by sleeping. There is no shell involved anywhere (no injection surface).
- **Minimal dependencies.** Do not add a crate when `std` suffices. `rmcp` is allowed only for MCP
  protocol/tool serving, not daemon IPC or debugger backend RPC. Deliberately excluded: `once_cell`, `nix`,
  `daemonize`, `portable-pty`/`conpty`, any broad TCP/RPC framework, any windows/winapi crate (use `std` raw
  FFI for the few Win32 calls). Use `std::sync::LazyLock` for one-time regex compilation and `std::os::*`
  for platform bits.
- **Keep Java sidecar dependencies isolated.** Java-only expression/evaluation dependencies belong in
  `sidecar/jdi` and are packaged into the Gradle fat jar `jdbg-jdi-sidecar.jar`. Do not add a Rust RPC
  framework or Rust JDWP stack for sidecar work unless the user explicitly approves that redesign.
- **One daemon per user; one in-flight command per session.** The per-session command `Mutex` is held across
  write+wait — `jdb` is line-oriented and cannot interleave commands. Different sessions run in parallel.
- **The daemon is the single writer** of the on-disk registry (atomic temp-in-same-dir + rename). The CLI only
  reads it (offline fallback). The in-memory session map is the source of truth while the daemon is alive.
- **Keep modules small and single-purpose.** If a file grows large it is doing too much — split it.
- **Use English for code comments and commit messages.** Source/test comments, Rustdoc, git commit messages, and
  PR/review comments written for the project history should be in English.

## jdb engine contract (the riskiest code — get these exactly right)

- **MANDATORY locale flags.** Always spawn `jdb` with `-J-Duser.language=en -J-Duser.country=US
  -J-Dfile.encoding=UTF-8`. On this machine `jdb` otherwise emits localized (Chinese) event banners
  (`Breakpoint hit:`, `Step completed:`, `Exception occurred:`, `The application exited`) that will NOT match
  the English regexes. Prompt detection is locale-independent (primary signal); forcing English makes the event
  banners reliable (secondary signal). **Omitting these flags silently breaks parsing.**
- **en_US thousands separator in line numbers.** Forcing US English locale causes jdb to format line numbers
  ≥1000 with comma separators (`line=3,956`). All line-number regexes use `[\d,]+` and strip commas before
  parsing to `u32`. This affects `RE_BREAKPOINT_OR_STEP`, `RE_FIELD_WATCH`, `RE_SOURCE_LINE`, and
  `parse_location_parens`. **Never use bare `\d+` for line numbers in jdb output.**
- **Timeout clears the buffer.** `read_until_prompt` uses `take_text()` (not `.clone()`) on timeout, so
  subsequent commands start with a clean buffer. Stale data in the buffer after timeout was a critical bug
  causing output misalignment and wrong line-number captures.
- **Normal commands always purge.** `execute()` calls `purge_pending()` before any Normal command regardless
  of current `RunState`. This clears channel residue from late-arriving bytes after timeouts or events.
- **"Nothing suspended." fast return.** In Blocking mode, if jdb responds with "Nothing suspended." + bare
  prompt (VM was already running, `cont`/`resume` is a no-op), the reader returns immediately rather than
  waiting for the full blocking timeout.
- **Attach deduplication.** `create_attach` rejects connections to a target (host:port) that already has a
  live session. Two jdb clients on the same JDWP port interfere (one's `kill` sends `resume` that unfreezes
  the other's breakpoint).
- **Piped stdio, not ConPTY.** Plain `std::process::Command` with piped stdin/stdout/stderr. ConPTY injects
  ANSI/cursor escapes that are harder to parse. (Keep ConPTY only as a documented future fallback if some JDK
  withholds the prompt on a pipe.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PieceOfFall/jdbg](https://github.com/PieceOfFall/jdbg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
