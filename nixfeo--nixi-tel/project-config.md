---
trigger: always_on
description: Guidance for working in this repository (humans and AI assistants alike).
---

# NIXI SBC — Contributor & Architecture Guide

Guidance for working in this repository (humans and AI assistants alike).
For usage docs see [README.md](README.md), [docs/API.md](docs/API.md) and
[docs/WEBRTC.md](docs/WEBRTC.md).

## What this is

**nixi.tel** is an API-first **Session Border Controller** in Rust for SIP
trunking and WebRTC. Full B2BUA call control, media anchoring with
transcoding, and a complete REST API — every piece of dynamic configuration
(users, trunks, DIDs, routes, ACLs, bans) is managed at runtime over HTTP
and persisted in an embedded SQLite store. No external database required.

MIT licensed. Runs in production; contributions welcome — see
[CONTRIBUTING.md](CONTRIBUTING.md).

## Workspace layout

| Crate | Role |
|-------|------|
| `crates/sbc-core` | SIP engine: transports, B2BUA, media (RTP/SRTP/DTLS/ICE), transcoding, auth, security |
| `crates/sbc-storage` | SQLite `ConfigStore` — source of truth for dynamic config |
| `crates/sbc-management` | axum REST API server (routes, SSE, auth middleware) |
| `crates/sbc-bin` | Binary entry point — wires the above together |
| `rsip-nixi/` | Vendored fork of [rsip](https://github.com/vasilakisfil/rsip) (SIP parsing) |

## Build & test

```bash
cargo build --workspace
cargo test --workspace          # ~470 tests
cargo clippy --workspace
```

**cmake is required** for the bundled Opus codec (`audiopus_sys`). If your
cmake is 4.x, export `CMAKE_POLICY_VERSION_MINIMUM=3.5` (the vendored
libopus declares an older minimum). The `rsip-nixi/` fork is
workspace-excluded and built as a path dependency.

Run locally:

```bash
cp config/sbc.toml.example config/sbc.toml   # edit realm, public IP, token
cargo run --release -- --config config/sbc.toml
```

## Architecture

### Config model

Static config (network listeners, media, logging) lives in the TOML file.
**Dynamic config** (SIP users, DIDs, trunks, routes, ACL rules, bans) lives
in the SQLite store and is the source of truth. On first boot, TOML seed
entries (`[security.sip_users]`, `[[trunks]]`, `[[dids]]`) are imported once
into the store, then ignored. Every API write goes to the store and is
applied to the live runtime immediately — no reload needed. `SIGHUP` and
`POST /api/v1/reload` re-hydrate the runtime from the store.

Key modules: `sbc/import.rs` (first-boot seed), `sbc/hydrate.rs`
(store → live managers), `config.rs` (TOML schema).

### Call flow

Inbound message → `sbc/mod.rs` pipeline: **ban → ACL → DoS → dispatch**.
INVITEs go through `sbc/invite_handler.rs` (routing, DID mapping, trunk
selection with failover, topology hiding); responses through
`sbc/response_handler.rs` (SDP rewriting, WebRTC/SRTP, session-timer 200s);
BYE/CANCEL/ACK/INFO/re-INVITE through `sbc/call_handler.rs`. The B2BUA
(`b2bua.rs`) holds per-call dialog state for both legs.

### Key files

| File | Role |
|------|------|
| `sbc/mod.rs` | Core struct, config, event loop, REGISTER, OPTIONS, pipeline |
| `sbc/invite_handler.rs` | INVITE routing, trunk failover, 407 retry, session-timer offer |
| `sbc/response_handler.rs` | Response relay, SDP, WebRTC/DTLS/SRTP, session-timer completion |
| `sbc/call_handler.rs` | BYE/CANCEL/ACK/INFO, re-INVITE, timeouts, graceful shutdown |
| `sbc/hydrate.rs` · `sbc/import.rs` | Store → runtime hydration / first-boot TOML seed |
| `sip_builder.rs` | Synthetic in-dialog requests (BYE/CANCEL/ACK/re-INVITE) from real dialog identity |
| `b2bua.rs` | B2BUA half-mode, dialog state, failover state, session timers |
| `events.rs` | `EventBus` → SSE `/api/v1/events` |
| `security/` | fail2ban banning, anti-IRSF destination rules, per-user limits |
| `routing/{trunk,router}.rs` | TrunkConfig, LCR, `route_request_candidates()` for failover |
| `media/rtp.rs` | Bidirectional RTP relay, STUN/DTLS demux, DTMF PT re-mapping, inactivity timeout |
| `media/{sdp,srtp_crypto,ice,dtls,stun}.rs` | SDP rewriting, SRTP, ICE, DTLS, STUN |
| `transport/{udp,tcp,tls,ws}.rs` · `transport/tls_connect.rs` | Listeners + real outbound TLS |
| `transcoding.rs` | Opus ↔ G.711 (PCMU/PCMA) with resampling |
| `topology.rs` | Via/Contact/Record-Route rewriting (RFC 3261) |
| `auth.rs` · `register.rs` | Digest auth (401/407, nonce); SIP registrar |
| `metrics.rs` · `storage.rs` · `dos.rs` · `acl.rs` | Prometheus, CDR, rate limiting, IP ACLs |
| `crates/sbc-management/src/{server,state,routes/}` | axum API server |

### Inactive / legacy modules

- `media/turn.rs`, `media/data_channel.rs` — feature-gated (`turn`,
  `data-channel`); TURN is expected to be an external coturn (see docs/WEBRTC.md)
- `tls_client.rs` — legacy simulated TLS, superseded by `transport/tls_connect.rs`
- `http_server.rs`, `api.rs` — legacy hand-rolled HTTP, superseded by the axum server
- `dialog/`, `transaction/` — state machines bypassed (B2BUA + stateless processing)

## REST API

Full reference in [docs/API.md](docs/API.md). All dynamic config is
SQLite-backed and applied to the runtime immediately. Highlights: CRUD for
users/DIDs/trunks/routes/ACL; `/api/v1/security/*` (bans, destination rules,
user limits); `GET /api/v1/events` (SSE); `GET /api/v1/export` (backup);
`DELETE /api/v1/calls/{uuid}`. `/health` and `/ready` are public; everything
else needs the bearer token (constant-time comparison).

## Deployment


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NIXFEO/NIXI.TEL](https://github.com/NIXFEO/NIXI.TEL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
