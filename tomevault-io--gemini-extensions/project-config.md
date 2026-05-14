---
trigger: always_on
description: > A peer-to-peer chat substrate that lets multiple Claude Code instances share the same context. See @README.md for the user story and demo flow; this file is for Claude.
---

## cc-connect

> A peer-to-peer chat substrate that lets multiple Claude Code instances share the same context. See @README.md for the user story and demo flow; this file is for Claude.

# cc-connect — agent guide

A peer-to-peer chat substrate that lets multiple Claude Code instances share the same context. See @README.md for the user story and demo flow; this file is for Claude.

## What this repo is (mental model)

- **Rust workspace** with five crates (`cc-connect`, `cc-connect-core`, `cc-connect-hook`, `cc-connect-mcp`, `cc-connect-tui`) plus a **Bun + React + Ink** chat panel under `chat-ui/`. Both build into `target/release/`.
- The substrate is **chat-as-context**: every Peer's Claude reads from a locally-replicated `~/.cc-connect/rooms/<topic>/log.jsonl`. The `UserPromptSubmit` hook (`cc-connect-hook`) injects unread messages into the next prompt. That's the magic.
- The MCP server (`cc-connect-mcp`) lets Claude write back into the room — `cc_send`, `cc_at`, `cc_drop`, etc.

## Read these before deep work

The canonical specs live in repo root, not in this file. Do not duplicate them here.

- @CONTEXT.md — Ubiquitous Language. **Use these terms verbatim** (Room, Ticket, Substrate, Peer, Host, Identity, Pubkey, Message, Hook, Cursor, Backfill, Session, Injection, Context). Never drift to "channel", "session", "client", "history", "memory" except when the term genuinely applies.
- [PROTOCOL.md](PROTOCOL.md) — wire spec, RFC 2119 keywords. Read on demand for anything touching gossip payloads, Tickets, Backfill RPC, file_drop, or on-disk layout. **Wire-format changes are breaking** — bump `v` and the ALPN.
- [SECURITY.md](SECURITY.md) — threat model. Read on demand before changing anything in `cc_drop` blocklists, hook injection paths, identity handling, or relay routing.
- [TODOS.md](TODOS.md) — known gaps and the v0.1 → v1.0 migration list. Read on demand when picking up unfinished work.
- `docs/adr/` — decisions already made. Don't re-litigate; if you must reopen, mark the contradiction explicitly.

## Commands you can't infer from the code

```bash
# Workspace build (Rust + chat-ui together — install.sh wraps this)
./install.sh                               # interactive, idempotent; sets up hook + MCP
cargo build --workspace --release          # Rust only
(cd chat-ui && bun install && bun run build)   # chat-ui → target/release/cc-chat-ui

# Tests
cargo test --workspace                     # Rust unit + integration
scripts/smoke-test.sh                      # end-to-end: spin two peers, check gossip
scripts/smoke-test-mcp.sh                  # MCP server tools
scripts/smoke-test-bg.sh                   # background host-daemon path
(cd chat-ui && bun test && bunx tsc --noEmit)

# Diagnostics
./target/release/cc-connect doctor         # verify install (hook entry, identity, perms)
```

`Cargo.lock` **is tracked on purpose** — this repo ships binaries and reproducible builds gate the v0.1 release. Do not gitignore it.

## Non-obvious gotchas

- **Vendored ed25519 / ed25519-dalek**. `Cargo.toml`'s `[patch.crates-io]` points at `vendored/ed25519` + `vendored/ed25519-dalek`. The published `ed25519-3.0.0-rc.4` is broken against current `pkcs8` (`Error::KeyMalformed` enum-variant break). Drop the patch only when upstream ships a working `ed25519-dalek`. See [TODOS.md](TODOS.md).
- **MSRV is 1.89** (`workspace.package.rust-version`). Driven by the iroh stack itself (`iroh@0.97`, `iroh-blobs@0.99`, `iroh-gossip@0.97`, `iroh-relay@0.97` all require 1.89). CI gates on this; install.sh actively `rustup update`s when the user has an older toolchain.
- **Hook trust boundary**. `cc-connect-hook` injects chat context only when `CC_CONNECT_ROOM` is set in its env (set by `cc-connect-tui` when it spawns the Claude PTY). Unrelated `claude` invocations on the same machine see nothing. Don't loosen this — it's the cross-process isolation guarantee in [SECURITY.md](SECURITY.md).
- **PID-based active-rooms discovery** lives at `/tmp/cc-connect-$UID/active-rooms/<topic>.active`, not under `~`. PIDs are per-machine; cloud-synced homes would collide. See `docs/adr/0003-pid-based-active-rooms-discovery.md`.
- **8 KB hook stdout budget.** `cc-connect-hook` keeps each `UserPromptSubmit` payload ≤ 8 KB so it stays inline; over that, Claude Code falls back to a 2 KB preview + persisted file. See `docs/adr/0004-hook-budget-and-graceful-overflow.md`.
- **`cc_drop` path blocklist** rejects `~/.ssh`, `~/.aws`, `.env*`, `id_rsa*`, `*.pem`, etc. Don't widen it without a SECURITY.md update; don't narrow it without an explicit threat-model justification.
- **iroh dependency pin** — `iroh 0.97`, `iroh-gossip 0.97`, `iroh-blobs 0.99`. The combo is non-trivial; bumping any one usually requires bumping all three together.

## Workflow

- **Plan before coding for protocol or hook work.** Anything touching wire format, identity, hook injection, or `cc_drop` semantics gets a Plan-mode pass first — these break compatibility silently if you guess.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomevault-io/gemini-extensions](https://github.com/tomevault-io/gemini-extensions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
