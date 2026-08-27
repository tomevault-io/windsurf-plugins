---
trigger: always_on
description: This document describes the architecture and the rules for anyone, human or agent, working on
---

# Agent Instructions

This document describes the architecture and the rules for anyone, human or agent, working on
Mesh Sync.
Read [HANDOFF.md](HANDOFF.md) alongside it: it records the findings behind these decisions, most
of which are not guessable from the code.

This document and HANDOFF.md stay the source of truth for the rules and the findings.
[docs/](docs/Home.md) is the map over the top of them: one note per feature, mechanism and head,
saying where each thing lives and which platforms actually have it.
Go there first to orient, come back here for what must never be broken.

## The Goal

- **Absolute privacy**: no cloud hosting, no third-party servers, no account.
- **Seamless sync**: devices stay in sync automatically whenever they are in range of each other.
- **Universal clipboard**: copy text or an image on any device, paste it on any other.

## Architecture

### 1. Identity comes first

Every device holds a persisted P-256 keypair.
Its identity is the SHA-256 fingerprint of that key, and that identity decides three separate
things: whether a peer is allowed to connect, which key traffic is sealed with, and which role
each device takes on a link.

**There is one key per connection**, not per pair and certainly not per mesh.
Each end mints an ephemeral P-256 keypair, announces it in the hello, and the session key mixes
two ECDH secrets through HKDF: the ephemeral one gives forward secrecy, the static one gives
authentication.
An attacker can complete the first with anybody, because it is unauthenticated by construction,
but not the second without a private key this device has paired with - so the two ends never
agree and AES-GCM refuses the payload.
This is the shape of Noise's `KK` handshake, deliberately, so it can be read against a known
pattern rather than assessed as an invention.

Both fingerprints are sorted into the salt so the two ends mix the same bytes in the same order.
Unsorted, they derive different keys and every payload fails to decrypt with nothing on the wire
to say why.

The key therefore belongs to the connection, which is what `PeerSession` is.
Disposing it is what makes the traffic unrecoverable, so a link that closes takes its key with
it.

A listener refuses any peer it has not paired with.
A stranger gets no further than a queue: showing the pairing code says somebody was invited, and
comparing the four-group fingerprint on both screens says it is the *right* somebody.
That second step is what closes the race an attacker on the same network could otherwise win.

### 2. Two transport tiers, neither of them a fallback

- **Bluetooth LE is the standing link.**
  Held continuously whenever a peer is in range.
  Carries text, presence and control frames at roughly 6.7 KB/s.
  Peers are found by scanning for the service UUID, so pairing carries no Bluetooth address and
  no OS-level bonding is used or needed.
- **Wi-Fi is raised on demand.**
  Length-prefixed TCP on port 45001.
  Carries anything, and is the only tier that carries images.

Wi-Fi is wanted **per peer**, when any of these hold for that device: the screen is on, a send is
holding it for that peer, that peer has asked for it, or nothing is carrying presence for it.
**That last one is load-bearing.** Without it, losing Bluetooth would leave a device with no link
at all, and inverting the tiers would have been a regression rather than an improvement.
It was one boolean for the whole device until v0.4, so a radio link to one peer dropped the socket
to every other.

A device holding something Bluetooth cannot carry sends a `ControlWakeWiFi` frame over the open
link, and its peer raises Wi-Fi in response.
This exists because a device cannot dial its peer on demand: either end may be the listener.

### 3. No fixed roles

Every device listens **and** dials on both tiers.

- **Wi-Fi**: when two devices dial each other at once they collide, and the link opened by the
  lower fingerprint survives.
  Both ends compute that from values they already exchanged, so there is no negotiation round trip.
- **Bluetooth**: GATT roles are genuinely asymmetric, so `BleRoleRules` decides them
  **capability first, fingerprint second**.
  Advertising is hardware-dependent on Android, so a device that cannot advertise must be the
  central whatever its fingerprint sorts to.
  The naive "lower fingerprint advertises" rule agrees on an arrangement neither device can
  perform.
  Each device **announces** its capability in both hellos since wire version 4, so the rule stops
  assuming `Both` for the peer - which is what made two devices that both cannot advertise sit
  waiting for each other.

### 4. One link object per peer, and no relaying

**`CoreLib.Transport.Fabric` is the whole connection layer, and every head runs on it.**

`MeshFabric` holds one `PeerLink` per paired device, and a `PeerLink` owns every route to that
device - a socket, a radio link this device opened, a radio link the peer opened. A route is one
small interface with one state machine, and `LinkSupervisor` reconciles what exists against what
`RoutePolicy` wants.

**The state machine is the design, not an implementation detail.** There is no transition into

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [x20surya/MeshSync](https://github.com/x20surya/MeshSync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
