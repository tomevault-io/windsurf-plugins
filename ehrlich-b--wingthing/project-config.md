---
trigger: always_on
description: `wt` runs AI agents sandboxed on your machine, accessible from anywhere. The primary use case is `wt egg <agent>` (sandboxed agent sessions) and `wt wing` (remote access via relay). Skills are a secondary feature.
---

# Wingthing

## What This Is

`wt` runs AI agents sandboxed on your machine, accessible from anywhere. The primary use case is `wt egg <agent>` (sandboxed agent sessions) and `wt wing` (remote access via relay). Skills are a secondary feature.

- `wt egg claude` -- run Claude Code in a per-session sandbox with PTY persistence
- `wt start` -- connect your machine to the relay, access from app.wingthing.ai
- `wt serve` -- relay server (web UI, WebSocket relay, skill registry), HTTP + SQLite

## Design Philosophy

**Curated > marketplace.** Skills live in `skills/` in this repo. They're reviewed, validated, and version-controlled. No storefront where anyone can publish prompt injections. Private skills go in `~/.wingthing/skills/`.

**Sandbox-first.** `internal/sandbox/` has Seatbelt (macOS) and user namespace/seccomp (Linux). The sandbox IS the permission boundary for egg sessions — agents get `--dangerously-skip-permissions` because the sandbox constrains them.

**Agent-agnostic.** Every skill works with every backend. `--agent ollama` for free local inference, `--agent claude` when you need it. The interface is stable; providers change behind it.

**Local-first.** Your machine, your keys, your data. No cloud dependency. Offline with ollama.

## Dogfooding

**Always use wingthing's own tools and infrastructure.** If wingthing can do something, use wingthing to do it. Don't shell out to external scripts or paid APIs when the equivalent exists (or should exist) in the codebase.

If you find yourself reaching for an external tool and wingthing _should_ handle it, that's a gap to fill in wingthing itself.

## Architecture

- `wt egg <agent>` -- spawns a per-session child process (`wt egg run`) with its own sandbox, PTY, and gRPC socket at `~/.wingthing/eggs/<session-id>/`
- `wt wing` -- WebSocket client that connects outbound to the relay, handles PTY sessions and encrypted tunnel requests, spawns eggs for each session
- `wt serve` -- relay server (web UI + WebSocket relay + skill registry), HTTP + SQLite. The relay is a dumb pipe for wing data -- it forwards encrypted blobs without reading them.
- **The relay knows NOTHING about wings except their IDs and public keys.** `GET /api/app/wings` returns a list of wing UUIDs. All wing metadata (hostname, platform, agents, projects, labels) comes from the wing itself via encrypted tunnel requests (`wing.info`). The frontend must cache this metadata in localStorage and show cached data on page load while probing wings in the background.
- `wt run` -- direct agent invocation for prompts and skills (the old `wt [prompt]`)
- `wt roost` -- combined relay + wing in one process for self-hosted deployments
- Agents are pluggable (claude, ollama, gemini, codex, cursor, opencode). `wt` calls them as child processes.
- All commands use direct store access via `store.Open(cfg.DBPath())`.

### Encrypted Tunnel Protocol

All wing data (directory listings, session history, audit recordings, egg configs, passkey assertions) flows through an E2E encrypted tunnel. The relay cannot read any of it.

| Message | Direction | Description |
|---------|-----------|-------------|
| `tunnel.req` | browser -> relay -> wing | Encrypted request: `{type, wing_id, request_id, sender_pub, payload}` |
| `tunnel.res` | wing -> relay -> browser | Encrypted response: `{type, request_id, payload}` |
| `tunnel.stream` | wing -> relay -> browser | Encrypted streaming: `{type, request_id, payload, done}` |

Inner message types (inside encrypted payload): `dir.list`, `wing.info`, `webrtc.offer`, `sessions.list`, `sessions.history`, `audit.request`, `egg.config_update`, `pty.kill`, `wing.update`, `passkey.auth`, `allow.list`, `allow.add`, `allow.remove`, `paths.list`, `paths.set`, `paths.add_member`, `paths.remove_member`

### Two Key Types, Two HKDF Domains

| Key | Lifecycle | HKDF info | Purpose |
|-----|-----------|-----------|---------|
| PTY session key | Per-session ephemeral X25519 | `"wt-pty"` | Terminal I/O encryption |
| Tunnel key | Persistent identity X25519 | `"wt-tunnel"` | All non-PTY wing data |

Browser identity key is stored in sessionStorage (ephemeral per tab, provides PFS). Passkey auth tokens are shared between PTY and tunnel, with configurable TTL via `auth_ttl` in wing.yaml. Wing restart revokes all sessions (in-memory cache).

### Wing ID Scheme (IMPORTANT — two different IDs)

Wings have TWO identifiers. Confusing them breaks session routing.

| ID | Field | Format | Lifecycle | Example |
|----|-------|--------|-----------|---------|
| **Machine ID** (`wing_id` / `WingID`) | `ConnectedWing.WingID`, API `wing_id` | 24-char hex (MongoDB-style) | Persistent, stored in `~/.wingthing/wing.yaml` | `1ae20a6b28854276b1514d14` |
| **Connection ID** (`id` / `ID`) | `ConnectedWing.ID`, registry map key | UUID prefix or random | Ephemeral, assigned on WebSocket connect | `a1b2c3d4` |

**The API (`/api/app/wings`) returns `wing_id` (machine ID).** The frontend uses `wing_id` everywhere. The `wings` map in `WingRegistry` is keyed by connection ID (`ConnectedWing.ID`).

**Lookup patterns:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ehrlich-b/wingthing](https://github.com/ehrlich-b/wingthing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
