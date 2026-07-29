---
trigger: always_on
description: **nim-libp2p** is a native Nim implementation of the [libp2p](https://libp2p.io) peer-to-peer networking stack. It is used in production by [Nimbus (Ethereum client)](https://github.com/status-im/nimbus-eth2) and other projects.
---

# nim-libp2p Coding Agent Instructions

## Project Overview

**nim-libp2p** is a native Nim implementation of the [libp2p](https://libp2p.io) peer-to-peer networking stack. It is used in production by [Nimbus (Ethereum client)](https://github.com/status-im/nimbus-eth2) and other projects.

- **Language**: Nim ( see `libp2p.nimble` and `.github/workflows/ci.yml` for currently supported versions)
- **License**: Apache 2.0 OR MIT
- **Version**: 1.15.3
- **Author**: Status Research & Development GmbH

---

## Repository Layout

```
nim-libp2p/
├── libp2p/           # Main source code
│   ├── switch.nim              # Core: manages connections, protocols, transports
│   ├── standard_switch.nim     # Default switch implementation
│   ├── builders.nim            # Builder pattern for switches
│   ├── peerid.nim / peerinfo.nim / peerstore.nim
│   ├── peeridauth/                 # Peer ID HTTP authentication (client/server)
│   ├── multiaddress.nim / multistream.nim
│   ├── crypto/                 # Cryptographic primitives
│   ├── muxers/                 # Stream multiplexers (mplex, yamux)
│   ├── protocols/              # P2P protocols
│   │   ├── connectivity/       # AutoNAT, DCUtR, Circuit Relay
│   │   ├── pubsub/             # GossipSub, FloodSub
│   │   ├── kademlia/           # Kademlia DHT
│   │   ├── perf/               # Performance measurement protocol
│   │   └── secure/             # Noise, Plaintext
│   ├── transports/             # TCP, QUIC, WebSocket, Tor, Memory
│   ├── stream/                 # Stream abstractions
│   ├── autotls/                # Automatic TLS certificate management (ACME)
│   ├── services/               # Auto-relay, hole punching, wildcard resolver
│   └── utils/                  # Utilities
├── tests/
│   ├── test_all.nim            # Main test runner
│   ├── libp2p/                 # Unit tests mirroring source structure
│   ├── integration/            # Integration tests (WebSocket, AutoTLS, peer ID auth)
│   └── interop/                # Cross-implementation interoperability tests
├── examples/                   # Tutorial and example applications
├── cbind/                      # C/CDDL FFI bindings (generated via nim-ffi)
├── docs/                       # Documentation
│   ├── README.md               # Documentation index
│   ├── development.md          # Setup and testing guide
│   ├── contributing.md         # Contribution guidelines
│   ├── compile_time_flags.md   # All compile-time flags documented
│   ├── common_hurdles.md       # Known issues and fixes
│   └── interop_hole_punching.md  # Hole punching interop test guide
├── tools/                      # Developer tools (dependency pinner, markdown runner, etc.)
├── libp2p.nim                  # Main entry point (re-exports public APIs)
├── libp2p.nimble               # Package manifest and build tasks
├── config.nims                 # Compiler configuration (warnings, style, memory)
└── Makefile                    # Top-level build targets
```

---

## Build & Test

### Setup
```sh
git clone https://github.com/vacp2p/nim-libp2p
cd nim-libp2p
make                 # Set up lowest supported dependency set
# Or: nix develop     # Nix-based dev environment
```

> **Note**: nimble 0.24.0+ is required for the min-version resolver. If using `nix develop`, the nix environment may not have a sufficiently recent nimble — in that case, run `nimble install nimble` inside the nix shell to get a newer version (typically installed to `~/.nimble/bin/nimble`).

### Running Tests
```sh
# Run all unit tests
make test

# Run tests matching a path substring
make test quic                         # all quic tests
make test transports/test_ws           # specific test file

# Run specific test suites
make test_multiformat_exts             # MultiFormat extension tests
make test_integration                  # Integration tests
```

### Faster Iteration (bypass nimble overhead)
```sh
nim c -r tests/test_all.nim
nim c -r -d:path=quic tests/test_all.nim
nim c -r tests/tools/test_multiaddress.nim
```

### Code Formatting
```sh
nimble install nph@v0.7.0   # Install formatter (once)
nimble format               # Format all code
```

### Logging / Debug
```sh
nim c -r -d:chronicles_log_level=debug examples/helloworld.nim
nim c -r -d:chronicles_enabled_topics:switch:TRACE,quictransport:INFO examples/helloworld.nim
```

---

## Compiler Configuration (`config.nims`)

Warnings are treated as errors. All of the following must pass cleanly:

```nim
switch("warningAsError", "BareExcept:on")
switch("warningAsError", "CaseTransition:on")
switch("warningAsError", "CStringConv:on")
switch("warningAsError", "ImplicitDefaultValue:on")
switch("warningAsError", "LockLevel:on")
switch("warningAsError", "ObservableStores:on")
switch("warningAsError", "ResultShadowed:on")
switch("warningAsError", "UnreachableCode:on")
switch("warningAsError", "UnreachableElse:on")
switch("warningAsError", "UnusedImport:on")
switch("warningAsError", "UseBase:on")
switch("hintAsError", "ConvFromXtoItselfNotNeeded:on")
switch("hintAsError", "DuplicateModuleImport:on")
--styleCheck: usages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vacp2p/nim-libp2p](https://github.com/vacp2p/nim-libp2p) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
