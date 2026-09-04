---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Rust-based network tunneling/proxy system called `pb-mapper` that allows exposing local services to clients over a public network. The project enables users to access their home services (like file transfer servers) from anywhere by creating secure tunnels through a public server.

The system uses one **pb-mapper** binary (`crates/pb-mapper-cli/src/bin/pb-mapper.rs`) with explicit role commands:

1. **`pb-mapper server`**: Central server that manages connections between local services and clients
   - Runs on port 7666 by default
   - Supports IPv4/IPv6 configuration
   - Manages service registration and client subscription mappings
   - Handles connection forwarding and keep-alive mechanisms

2. **`pb-mapper register`**: Registers local services with the central server
   - Exposes local TCP/UDP services to the public server
   - Supports encryption codec for secure communication
   - Configurable via environment variables and command-line arguments

3. **`pb-mapper connect`**: Connects to registered services through the central server
   - Subscribes to remote services and creates local listening endpoints
   - Supports both TCP and UDP protocols

4. **`pb-mapper status`**: Queries remote IDs and registered service keys

5. **`pb-mapper admin`**: Administrator operations against a running server —
   issuing, listing, and revoking temporary credentials, rotating the
   administrator key, and listing services and connections

6. **UI Module** (`ui/`): Flutter graphical interface
   - Replaces all CLI functionality with a user-friendly GUI
   - Calls into Rust through raw `dart:ffi` against the `pb-mapper-ffi` crate
   - Provides comprehensive service management interface

The system works by creating a bridge between local services and remote clients through a public server, enabling access to services behind NAT/firewalls.

## Code Architecture

### Project Structure

The root `Cargo.toml` is a virtual manifest; every crate lives under `crates/`,
except the FFI cdylib, which sits next to the Flutter code that loads it.

```
pb-mapper/
├── crates/
│   ├── pb-mapper-core/     # Bottom layer: checksum, config, conn_id, error,
│   │                       # addr, codec, timeout, durable_file, DataLenType
│   ├── pb-mapper-auth/     # Credential lifecycle, persistence, timing wheel
│   ├── pb-mapper-protocol/ # Message framing, v2 secure sessions, forwarding
│   ├── pb-mapper-server/   # Central relay server, plus the task manager
│   ├── pb-mapper-client/   # Both tunnel ends: `register` and `connect`
│   ├── pb-mapper-testkit/  # Test support: a complete e2e tunnel, for any test file
│   └── pb-mapper-cli/      # The `pb-mapper` binary, integration tests, examples
├── ui/                    # Flutter UI, talking to Rust over dart:ffi
│   ├── lib/               # Flutter application code
│   │   ├── l10n/          # ARB sources and generated AppLocalizations
│   │   └── src/ffi/       # The Dart side of the FFI boundary
│   ├── native/pb_mapper_ffi/  # C ABI crate (a workspace member)
│   └── test/              # Widget tests
├── docker/                # Docker deployment configuration
└── services/              # Systemd service files
```

The dependency graph is a DAG, and the layering is what the crate split
encodes:

```
pb-mapper-cli          pb-mapper-ffi
      │                      │
      └────┬─────────────────┤
           ▼                 ▼
    pb-mapper-server   pb-mapper-client   (peers: no reference either way)
           └──────┬──────────┘
                  ▼
          pb-mapper-protocol
                  ▼
            pb-mapper-auth
                  ▼
            pb-mapper-core
```

Note that the binary is still named `pb-mapper`, discovered from
`src/bin/pb-mapper.rs` inside `pb-mapper-cli`. The release workflows, both
Dockerfiles, and the install scripts hardcode that name, and `cargo build --bin
pb-mapper` resolves it from the workspace root regardless of the crate name.
Likewise `pb-mapper-ffi` keeps its package name, because it determines the
`libpb_mapper_ffi.{so,dylib,a}` / `pb_mapper_ffi.dll` filenames that the Dart
loader, two CMakeLists, four xcconfigs, and the release-ui hash checks expect.

### Core Modules

#### Rust Backend (`crates/`)
- **`pb-mapper-core/`**: The bottom layer; depends on no other crate here
  - `checksum.rs`: The process credential, and the framing checksum over `datalen`
  - `config.rs`: Environment configuration and address resolution entry points
  - `conn_id.rs`: Connection ID types
  - `error.rs`: The shared error type, plus the `snafu_error_*` macros
  - `addr.rs`: Address resolution; custom DNS servers on the async path
  - `codec.rs`: AES-256-GCM encrypt/decrypt
  - `timeout.rs`: `RetryBackoff`
  - `durable_file.rs`: Atomic replace and parent-directory fsync
  - `test_support.rs`: `PROCESS_CREDENTIAL_TEST_LOCK`, shared across crates' tests
  - `lib.rs`: `DataLenType`, which lives here so `checksum` and `error` can name it

- **`pb-mapper-auth/`**: The credential subsystem, and the largest one

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [acking-you/pb-mapper](https://github.com/acking-you/pb-mapper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
