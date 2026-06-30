---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Shellshare is a live terminal broadcasting tool that allows users to share their terminal session via web links. The project is a complete Rust rewrite providing both client and server in a single binary.

## Build & Development Commands

```bash
# Build
make build                     # Same as above

# Lint (pedantic clippy lints are enabled)
make lint                      # Both clippy + check

# Run locally
cargo run -- server            # Start server on 0.0.0.0:3000
cargo run -- server --port 8080 --host 127.0.0.1
cargo run -- --server http://localhost:3000  # Run client
cargo run -- serve             # Share via a local server on localhost:3000

# E2E tests (Python + Playwright)
# Requires a release binary: the suite starts its own servers from it
cargo build --release
cd e2e && uv sync && uv run pytest -n 10
```

## Architecture

**Dual-mode binary**: `shellshare` operates as client (default) or server (`shellshare server`). `shellshare serve` combines both: it boots the embedded server on a background thread (default `localhost:3000`, configurable via `--host`/`--port`) and runs the client against it, sharing the terminal with no external server. Both `serve` and `server` accept `--tunnel` (`src/tunnel.rs`): it spawns the user's pre-installed `cloudflared` against the local server, waits for the `https://*.trycloudflare.com` URL from its stderr banner, and uses it as the share link (the broadcaster still talks to localhost); missing cloudflared is a fatal error pointing at the install docs, and the tunnel process dies with shellshare.

**Scripting surface**: `shellshare exec -- <cmd>` runs one command in the PTY (instead of a shell), broadcasts it, and exits with the command's exit code. The global `--json` flag switches stdout to newline-delimited JSON events: first one with event `sharing` (parse its `url` field), last `{"event":"end","exit_code":N}` (errors stay on stderr as `ERROR: ...`). This contract is documented in `AGENTS.md` and `public/llms.txt` and covered by `e2e/test_agents.py` - the three must stay in lockstep.

### Client (`src/cli/`)
Multi-threaded design ensures network latency never blocks terminal display:
- **PTY reader thread**: Captures shell output, displays locally, sends to the sender thread
- **Sender thread**: Owns the WebSocket transport; coalesces whatever is queued and sends immediately (no pacing - frames are cheap)
- **Stdin forwarder thread**: Routes user input to PTY
- **Signal handler** (Unix): Handles SIGWINCH for terminal resize

Key files:
- `mod.rs`: Entry point, room ID generation, server URL handling, terminal size. Connecting the transport claims the room, so auth failures surface before the shell spawns
- `script.rs`: PTY lifecycle, raw terminal mode, shell spawning
- `ws.rs`: WebSocket transport. Binary frames carry raw terminal bytes; JSON text frames carry control messages (`size`, `delete`). Reliability: output stays in a bounded replay buffer until the server acks it (`{"ack": n}`, cumulative per-connection bytes); on failure the client reconnects with backoff and replays everything unacked (at-least-once delivery). Only authorization errors are fatal
- `crypto.rs`: end-to-end encryption, on by default (opt out with `--disable-encryption`). Every output chunk is sealed into a self-delimiting AES-256-GCM record (`[u32 BE len][nonce][ciphertext+tag]`) as it enters the replay buffer, so acks/replay operate on ciphertext and the server stays an opaque relay (zero server changes - it never knew about encryption). The key is HKDF-derived from this machine's id and the room name (so a named room keeps one reusable share link across restarts; nothing is written to disk) and rides only in the link's `#fragment`. The `size` message carries `encrypted: true` so the viewer knows whether to decrypt (via WebCrypto, needs https or localhost) or render plaintext, and shows an explanatory notice when an encrypted link's fragment is missing/invalid/wrong or the context is insecure. `--disable-encryption` broadcasts plaintext for viewers on plain HTTP (a classroom LAN), where browsers have no WebCrypto. Record format must stay in lockstep with `templates/room.html`. Threat model (honest-but-curious server serving the unmodified page; key secrecy rests on the high-entropy machine id; metadata/timing still visible) is documented in `crypto.rs`

### Server (`src/server/`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vitorbaptista/shellshare](https://github.com/vitorbaptista/shellshare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
