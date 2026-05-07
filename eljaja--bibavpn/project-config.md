---
trigger: always_on
description: This document is the fast-path briefing for humans and AI agents contributing
---

# BibaVPN — context for agents and developers

This document is the fast-path briefing for humans and AI agents contributing
to BibaVPN. It covers what the project is, how the code is laid out, the CLI
knobs that actually matter on the wire, and the workflows you should follow
before proposing a change.

For on-wire byte layouts see **[PROTOCOL.md](PROTOCOL.md)**. For install / run
walkthroughs see **[README.md](README.md)**. For contributor etiquette see
**[CONTRIBUTING.md](CONTRIBUTING.md)**.

## Contents

- [What the project is](#what-the-project-is)
- [Repository layout](#repository-layout)
- [`bibavpn` crate modules](#bibavpn-crate-modules)
- [Biba v3 wire (short)](#biba-v3-wire-short)
- [BibaV2.1 and transport knobs](#bibav21-and-transport-knobs)
- [Build and run (local)](#build-and-run-local)
- [Docker / Compose gotcha](#docker--compose-gotcha)
- [Scripts](#scripts)
- [UDP design note (agents)](#udp-design-note-agents)
- [Testing and benchmarks](#testing-and-benchmarks)
- [Security](#security)
- [Guidelines for agents](#guidelines-for-agents)
- [BibaV1.2 stealth (status vs BibaV4)](#bibav12-stealth-status-vs-bibav4)
- [Scenarios that were validated](#scenarios-that-were-validated)

## What the project is

**BibaVPN** is a proxy stack: local **SOCKS5** (TCP `CONNECT` and
`UDP ASSOCIATE`) and optional **HTTP CONNECT** → **TLS + WebSocket** → remote
entry server → outbound **TCP or UDP** to the Internet.

The tunnel crypto on supported release lines is **Biba v3**: shared **PSK**,
opaque variable-length HELLO/ACK, ChaCha20-Poly1305, domain-separated KDF, sealed
control opcodes, and v3-style inner UDP records.

The **1.2.x** line still uses the **Biba v3** PSK wire on the inside; layered on
top are **BibaV1.2 “stealth”** options (TLS profile labels, padding modes, timing,
decoys, optional multi-WSS, optional BoringSSL outer TLS). The long-term product
spec **BibaV4** (see [PROTOCOL.md — BibaV4](PROTOCOL.md#bibav4-v120-target-specification))
may still change inner opcodes or framing — treat that section as roadmap where
it goes beyond current v3.

**BibaV2.1** transport knobs (WS ping, frame-size cap, custom headers, early noise)
sit on the WebSocket path. **BibaV1.2** adds `PadMode::Adaptive`, `--stealth-profile`,
`--fingerprint` / TLS resolution order, WebSocket jitter ranges, parallel outer
WSS sessions (`--ws-parallel`), idle-triggered decoys, server delayed-ACK / RTT
mask, and an optional **BoringSSL** build (`--features boring-tls`, `--tls-stack boring`).

**TCP — default:** many SOCKS connections share **one or more** TLS+WSS sessions
(**TCP mux** in `tcp_mux.rs`). With `--ws-parallel 2..=4`, the client opens that
many full v3 sessions (each with `MUX_OPEN`) and **round-robins** new streams
across them (`TcpMuxSessionPool::pick`). After v3 HELLO/ACK and sealed **AUTH**,
the client sends `MUX_OPEN`; per-target opens use mux records (stream id, flags,
payload) inside padded frames, with window-based flow control. Use `--no-mux` for
legacy **one WSS per SOCKS CONNECT** (`OPEN` + binary loop). **REALITY** mode uses
the same **`--ws-parallel` 1..=4** pattern: each outer link runs TLS + WSS +
REALITY (X25519) handshake, then `MUX_OPEN`; the pool **round-robins** new streams
(`connect_reality_tcp_mux_handle` in `local_client.rs`).

**UDP** (e.g. DNS via SOCKS5 UDP) uses a **separate** shared WSS:
`UDP_MUX_OPEN` (`protocol.rs`), then v3 **`0x05` UDP_REQ** / **`0x06` UDP_REP**
records (`udp_mux.rs`). Same TLS/WebSocket fingerprint class as TCP from the
client to the VPS.

**HTTP on the TLS port:** non-WebSocket requests are served as **camouflage**
(`incoming.rs`, `camouflage.rs`): nginx-style responses, optional
`--camouflage-dir` static files, or `--camouflage-url` (`http://host:port`
only — plaintext to origin).

**DPI-oriented options:** `--pad-mode adaptive|random|http-buckets`,
`--dummy-interval-secs` (idle empty padded frames on the tunnel; mux may share
several outer connections when `--ws-parallel` is between 2 and 4), **TLS profile** via
`--fingerprint` / `--tls-profile` (priority rules in `client_policy.rs` —
default client label **Chrome 132+** when nothing else applies), optional
`--stealth-profile default|balanced|aggressive` (fills pad/jitter/decoy/idle
thresholds when explicit flags are absent), **WebSocket send jitter** (`--ws-jitter-min/max-ms`
or legacy uniform delay), **idle decoy** HTTPS GETs when the mux is quiet longer
than `--idle-decoy-secs` (merged with preset; **10 s** in balanced/aggressive
presets unless overridden) — `activity.rs` + `decoy_traffic.rs`, client-only
parallel decoy `--decoy-gets` (+ interval and paths), `stealth_v12` decoy modes,
`stealth.rs` WebSocket upgrade header shape per `TlsClientProfile`. **Server:**
`--server-ack-delay-*-ms`, `--rtt-mask-jitter-ms`, and optional `--ack-profile
balanced|aggressive` when the explicit millisecond args are all zero
(`ServerRttDefaults` in `stealth_v12.rs`). **Outer TLS engine:** `rustls` (default,
`biba` cipher/ALPN hints) or **BoringSSL** (`cargo build -p bibavpn --features boring-tls`,
client `--tls-stack boring`); **certificate pinning** is not supported on the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eljaja/BibaVPN](https://github.com/Eljaja/BibaVPN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
