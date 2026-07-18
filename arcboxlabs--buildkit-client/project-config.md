---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Rust client library and CLI for interacting with BuildKit (moby/buildkit) to build container images via gRPC. Implements the complete BuildKit session protocol including bidirectional streaming, file synchronization, and HTTP/2-over-gRPC tunneling.

## Essential Commands

### Setup & Build
```bash
# First-time setup: initialize proto files
./scripts/init-proto.sh

# Start BuildKit daemon and local registry
docker-compose up -d

# Build project
cargo build --release

# Quick health check
cargo run -- health
```

### Development
```bash
# Build with logging
RUST_LOG=info cargo run -- local -c examples/test-dockerfile -t localhost:5000/test:latest

# Trace session protocol debugging
RUST_LOG=info,buildkit_client::session::grpc_tunnel=trace cargo run -- local -c . -t test:latest

# Session and fsutil protocol debugging
RUST_LOG=info,buildkit_client::session=debug timeout 25 cargo run -- local -c . -t test:latest
```

### Testing

#### Unit Tests (no BuildKit required)
```bash
# All unit tests
cargo test --lib
cargo test --test builder_test
cargo test --test session_test
cargo test --test progress_test

# Single test with output
cargo test test_platform_parse -- --nocapture
```

#### Integration Tests (requires BuildKit)
```bash
# Start BuildKit first
docker run -d --rm --privileged -p 1234:1234 moby/buildkit:latest --addr tcp://0.0.0.0:1234

# Run integration tests
cargo test --test integration_test -- --test-threads=1

# Run GitHub repository tests
GITHUB_TOKEN=your_token cargo test --test integration_test github -- --test-threads=1

# Using test script
./scripts/test.sh all          # All tests
./scripts/test.sh unit         # Unit tests only
./scripts/test.sh integration  # Integration tests
./scripts/test.sh github       # GitHub tests
```

#### Test Utilities (Makefile)
```bash
make -f Makefile.test test              # Unit tests
make -f Makefile.test test-integration  # Integration tests
make -f Makefile.test test-github       # GitHub tests
make -f Makefile.test coverage          # Coverage report
make -f Makefile.test bench             # Benchmarks
```

### Quality Checks
```bash
cargo fmt            # Format code
cargo clippy         # Lint
cargo bench          # Run benchmarks
```

## Architecture

### High-Level Data Flow

```
BuildKitClient.build(config)
  ↓
1. Create Session with UUID
2. Add FileSync service (for local builds)
3. Session.start() → Opens bidirectional gRPC stream
4. HTTP/2 tunnel starts inside session BytesMessage stream
  ↓
5. Prepare SolveRequest with:
   - Frontend: "dockerfile.v0"
   - Context: "input:session-{uuid}:context" (local) or git URL (GitHub)
   - Session metadata headers (X-Docker-Expose-Session-*)
  ↓
6. control.solve(request) → BuildKit begins build
  ↓
7. BuildKit calls back through HTTP/2 tunnel:
   - DiffCopy: Stream build context files
   - Credentials: Get registry auth
   - Health: Check session alive
  ↓
8. DiffCopy protocol:
   a. Server sends STAT packets (file metadata)
   b. Client sends REQ packets (requests file by ID)
   c. Server sends DATA packets (file content)
   d. Both send FIN when complete
  ↓
9. BuildKit completes build and pushes image
10. Return BuildResult with digest
```

### Critical Components

#### 1. Session Protocol (`src/session/mod.rs`)
- Orchestrates bidirectional gRPC stream with BuildKit
- Manages file sync and auth services
- Generates session metadata headers required by BuildKit
- **Key headers**: `X-Docker-Expose-Session-Uuid`, `X-Docker-Expose-Session-Grpc-Method`

#### 2. HTTP/2-over-gRPC Tunnel (`src/session/grpc_tunnel.rs`)
**Most complex component** - Implements a complete gRPC server inside a gRPC stream:

```
BuildKit Control.Session stream (outer)
  ↓ BytesMessage containing HTTP/2 frames
h2 server (inner)
  ↓ HTTP/2 frames → gRPC calls
Route to handlers:
  - /moby.filesync.v1.FileSync/DiffCopy → bidirectional stream
  - /moby.filesync.v1.Auth/Credentials → unary
  - /grpc.health.v1.Health/Check → unary
```

**gRPC Message Framing**:
- Each message: `[compression(1)] + [length(4 BE)] + [protobuf payload]`
- Must parse frame boundaries from buffered BytesMessage chunks
- Must add 5-byte prefix when sending responses

#### 3. DiffCopy File Sync (`src/session/diffcopy.rs`)
Implements fsutil's bidirectional streaming protocol for file transfer.

**Module Organization**: DiffCopy protocol is now in a separate module for better code organization.
The `grpc_tunnel.rs` delegates DiffCopy requests to this module via `diffcopy::handle_diff_copy_stream()`.

**Server → Client (sending files):**
1. Walk directory tree recursively
2. Send STAT packet for each entry (file/dir) with sequential ID (0, 1, 2...)
3. Send final empty STAT packet (no stat field) to signal end of listing
4. Wait for REQ packets from client

**Client → Server (requesting files):**
1. Receive all STAT packets
2. Send REQ packet with file ID for files that need data
3. Receive DATA packets for the file
4. Empty DATA packet signals end of that file
5. Send FIN packet when done requesting

**Server → Client (responding to REQs):**
1. Receive REQ packet with ID

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arcboxlabs/buildkit-client](https://github.com/arcboxlabs/buildkit-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
