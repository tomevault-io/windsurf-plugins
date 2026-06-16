---
trigger: always_on
description: A pure Go implementation of the OpenVPN3 client protocol — the same protocol
---

# Claude context for go-openlawsvpn

## What this repo is

A pure Go implementation of the OpenVPN3 client protocol — the same protocol
used by openvpn3-core (C++). It is a clean-room Go implementation based on the
OpenVPN protocol specification, with openvpn3-core source consulted only as a
historical reference. `go build` / `gomobile bind` produce a fully static binary
and an Android `.aar` with no C toolchain.

This repo **is** the engine. The old C++/openvpn3-core stack it once aimed to
replace is now fully retired and archived (see "Status" below).

## Parent project context

**openlawsvpn** is an open-source AWS Client VPN client with SAML/SSO support.
This repo is the current engine for all platforms:
- Linux CLI + daemon + GTK4 GUI: this repo (`cmd/cli`, `cmd/daemon`, `gui-gtk/`)
- Android client: https://github.com/openlawsvpn/openlawsvpn-android-go (consumes the `.aar`)
- Website: https://openlawsvpn.com

Consumers pin the engine via a `go-openlawsvpn.version` file and consume the
gomobile `.aar` produced by `aar.yml`.

### Status (verify version with `git tag --sort=-v:refname | head -1`)

The protocol is fully implemented and tested against a real AWS Client VPN
endpoint. Working end-to-end on Linux (CLI + daemon + GTK4 GUI) and Android
(via the gomobile `.aar`). Current tag: **v1.1.4**. RPM built by COPR `vorona/openlawsvpn` (Fedora 43/44/rawhide,
x86_64/aarch64/ppc64le). AUR package: `openlawsvpn` (x86_64/aarch64/powerpc64le);
AUR releases use `pkg/x.y.z-N` tags (separate from `v*` library tags).

### Retired / archived — do NOT treat as current

- `openlawsvpn/openlawsvpn` (archived) — old C++/openvpn3-core engine + Linux CLI.
- `openlawsvpn/openlawsvpn-android` (archived) — old Kotlin+JNI/NDK app.

There is no longer a `libopenlawsvpn` C library, no JNI, no NDK, and no
openvpn3-core dependency anywhere in the shipping stack. The C API and C++
reference sections below are kept only as protocol/porting background.

## Why Go

- `CGO_ENABLED=0` — fully static binary, zero native dependencies
- `gomobile bind` — produces `.aar` for Android without NDK/CMake
- Single codebase covers Linux CLI, Android, future iOS
- F-Droid compatible — no prebuilt blobs, `./gradlew assembleRelease` is self-contained
- Easier auditing, fuzzing (`go test -fuzz`), and contribution

## Goals (in priority order)

1. **Correctness over completeness** — every packet exchange must be byte-exact
   with what openvpn3-core expects. The mock server is the oracle.
2. **Android via gomobile** — `gomobile bind` produces an `.aar` that drops into
   `openlawsvpn-android-go` with no NDK changes.
3. **Linux static binary** — `CGO_ENABLED=0 go build` produces a binary with
   zero runtime dependencies.
4. **F-Droid compatibility** — no download-at-build-time, no prebuilt blobs.

## Non-goals

- OpenVPN server implementation
- OpenVPN 2.x legacy static-key mode
- Windows support
- WireGuard or other protocols

## SAML / CRV1 flow (the AWS-specific part)

AWS Client VPN uses a non-standard SAML challenge called CRV1:

```
Phase 1:
  Client → Server: TLS ClientHello + OpenVPN HARD_RESET
  Server → Client: AUTH_FAILED,CRV1:R,<state_id>::<saml_url>
  (connection pauses)

  App opens saml_url in browser.
  Browser → http://127.0.0.1:35001 : POST SAMLResponse=<base64>
  (AWS hardcodes ACS URL to 127.0.0.1:35001)

Phase 2:
  Client → Server: new TLS session with username="CRV1::<state_id>::<saml_token>"
  Server → Client: PUSH_REPLY with ifconfig, route, etc.
  Tunnel is up.
```

Key detail: AWS hardcodes AssertionConsumerServiceURL = http://127.0.0.1:35001.
This is true across all AWS regions and IdPs (Okta, Azure AD, Google Workspace).

The ACS server now lives in this repo's `auth/saml` package (Go). The old Kotlin
reference implementation in the archived `openlawsvpn-android` repo is obsolete.

## OpenVPN3 protocol reference

Key concepts an AI agent must know:

**Control channel (reliable transport)**
- Runs over TLS, but TLS runs inside OpenVPN's own reliable layer (not raw TCP TLS)
- Packet structure: [opcode (1 byte)][key_id (3 bits)][peer_id (24 bits)][packet_id (32 bits)][ack_array][payload]
- Opcodes: P_CONTROL_HARD_RESET_CLIENT_V2=0x38, P_ACK_V1=0x28, P_CONTROL_V1=0x20, P_DATA_V2=0x09
- Reliable layer: sequence numbers + sliding window + retransmit (matches reliable.hpp in openvpn3-core)
- TLS bytes are fragmented across P_CONTROL_V1 packets, reassembled in order

**Key derivation**
- After TLS handshake, both sides derive data channel keys using OpenVPN PRF:
  `key_material = PRF(master_secret, "OpenVPN master secret", client_random + server_random)`
- PRF is HMAC-SHA256-based, NOT the standard TLS PRF
- Produces 64 bytes: first 32 = cipher key, last 32 = HMAC key (for CBC mode)
- For GCM modes: only cipher key used, HMAC key unused

**Data channel**
- P_DATA_V2: [0x09 | key_id][peer_id (3 bytes)][iv (12 bytes for GCM)][ciphertext+tag]
- AES-256-GCM: IV = packet_id (32-bit counter, big-endian, zero-padded to 12 bytes) XOR implicit IV
- Replay protection: sliding window on packet_id (32-bit counter per session key)
- Key renegotiation: every 3600s or 100MB (configurable via reneg-sec/reneg-bytes in .ovpn)

**PUSH_REPLY parsing**
- After auth: server sends PUSH_REPLY with comma-separated options

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openlawsvpn/go-openlawsvpn](https://github.com/openlawsvpn/go-openlawsvpn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
