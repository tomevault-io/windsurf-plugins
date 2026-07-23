---
trigger: always_on
description: Rust workspace implementing secure peer-to-peer credential sharing over a WebSocket relay with end-to-end encryption via the Noise Protocol (NNpsk2 pattern). The relay server is zero-knowledge and cannot decrypt traffic.
---

# Agent Access SDK - Claude Code Configuration

Rust workspace implementing secure peer-to-peer credential sharing over a WebSocket relay with end-to-end encryption via the Noise Protocol (NNpsk2 pattern). The relay server is zero-knowledge and cannot decrypt traffic.

## Overview

### What This Project Does
- Enables secure, encrypted credential retrieval between a trusted device (UserClient) and an untrusted device (RemoteClient) through a zero-knowledge WebSocket relay
- Ships two binaries: `aac` (CLI for credential operations) and `ap-relay` (WebSocket relay server)
- Consumed as a library (`ap-client`) by integrators building custom credential-sharing workflows

### Key Concepts
- **Zero-knowledge relay** — the relay server authenticates clients by identity fingerprint but never sees plaintext credentials
- **Noise NNpsk2** — a Noise Protocol pattern providing encryption with optional pre-shared key authentication; the "NN" means neither side presents a static key during the handshake, and "psk2" injects a PSK after the second message
- **IdentityFingerprint** — SHA256 hash of a client's public key (32 bytes / 64 hex chars), used as a stable addressing identifier in the relay
- **HandshakeFingerprint** — 6-character hex string derived from transport keys (`SHA256(r2i_key || i2r_key)[0..3]`), used for out-of-band MITM verification
- **Rendezvous code** — 9-character alphanumeric code (format `ABC-DEF-GHI`, 36^9 entropy) with 5-minute TTL for initial pairing
- **PSK token** — `<64-hex-psk>_<64-hex-fingerprint>` (129 chars) for pre-authenticated pairing without rendezvous
- **PskId** — `SHA256(psk)[0..8]` as hex (16 chars), a non-secret identifier for PSK lookup
- **Session resumption** — cached `PersistentTransportState` (CBOR) allows reconnection without re-handshaking; transport auto-rekeys every 24 hours
- **Event-response model** — clients communicate via `tokio::sync::mpsc` channels; clients emit notifications (fire-and-forget) and requests (requiring a `oneshot` reply)

---

## Architecture & Patterns

### System Architecture

```
    Remote Device                          User Device
   (untrusted)                            (trusted)
        |                                      |
   RemoteClient                          UserClient
        |                                      |
   +----------+                         +----------+
   | ap-noise |  Noise NNpsk2 E2E       | ap-noise |
   |(handshake|<----------------------->|(handshake |
   |+transport|  XChaCha20-Poly1305     |+transport)|
   +----+-----+                         +-----+----+
        |                                      |
   ap-relay-client                      ap-relay-client
        |         WebSocket (JSON)             |
        +--------------+  +--------------------+
                       v  v
                +--------------+
                |   ap-relay   |  Zero-knowledge relay
                |  (WebSocket  |  Auth + Rendezvous +
                |   server)    |  Message routing
                +--------------+
```

### Code Organization

```
crates/
├── ap-cli/                  # CLI binary ("aac")
│   └── src/
│       ├── main.rs
│       ├── command/         # connect, listen, connections, run
│       │   ├── mod.rs       # Cli struct, Commands enum, dispatch
│       │   ├── connect.rs   # Remote-client connect flow
│       │   ├── listen.rs    # User-client listen flow
│       │   ├── connections.rs # list/clear cached sessions
│       │   ├── run.rs       # Fetch credential -> exec command
│       │   ├── output.rs    # OutputFormat (text/json)
│       │   ├── tui.rs       # Interactive ratatui TUI
│       │   ├── tui_tracing.rs
│       │   └── util.rs
│       ├── providers/
│       │   └── bitwarden.rs # bw CLI credential lookup
│       └── storage/
│           ├── session_storage.rs  # File-backed SessionStore
│           └── identity_storage.rs # File-backed IdentityProvider
├── ap-client/               # High-level client library
│   └── src/
│       ├── lib.rs
│       ├── clients/
│       │   ├── remote_client.rs  # RemoteClient + notifications/requests
│       │   └── user_client.rs    # UserClient + notifications/requests
│       ├── traits.rs        # SessionStore, IdentityProvider, AuditLog
│       ├── relay.rs         # RelayClient trait
│       ├── types.rs         # CredentialData, ConnectionMode, PskToken, etc.
│       ├── memory_session_store.rs
│       ├── error.rs         # ClientError
│       └── compat.rs
├── ap-noise/                # Noise Protocol + transport encryption
│   └── src/
│       ├── lib.rs
│       ├── handshake.rs     # InitiatorHandshake, ResponderHandshake
│       ├── transport.rs     # MultiDeviceTransport (encrypt/decrypt)
│       ├── ciphersuite.rs   # Classical vs PQ ciphersuite selection
│       ├── persistence.rs   # PersistentTransportState
│       ├── packet.rs        # HandshakePacket, TransportPacket
│       ├── psk.rs           # Psk, PskId generation
│       ├── symmetric_key.rs # SymmetricKey (ZeroizeOnDrop)
│       └── error.rs         # NoiseProtocolError

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitwarden/agent-access](https://github.com/bitwarden/agent-access) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
