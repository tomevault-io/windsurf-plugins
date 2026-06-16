---
trigger: always_on
description: Pure-Rust SMB2/3 client library with pipelined I/O. No C dependencies, no FFI. Single crate, async, runtime-agnostic.
---

# smb2

Pure-Rust SMB2/3 client library with pipelined I/O. No C dependencies, no FFI. Single crate, async, runtime-agnostic.

## Quick commands

- `just`: Fast checks: format, lint, test, doc (~2s)
- `just check-live`: Fast checks + integration tests on real servers (~6s)
- `just fix`: Auto-fix formatting and clippy warnings
- `just check-all`: Include MSRV check, security audit, and license check
- `just test-consumer`: Consumer integration tests (needs Docker, ~30s)
- `cargo test`: Run unit tests (mock transport, no server needed)
- `just fuzz <target> [duration]`: Fuzz a single parse entry point (nightly, cargo-fuzz)
- `just fuzz-seeds`: Regenerate the committed `fuzz/corpus/` seeds

## Project structure

```
src/
  lib.rs                  # Public API exports
  error.rs                # Error types, NTSTATUS mapping

  pack/                   # Binary serialization (cursor-based)
    mod.rs                # ReadCursor, WriteCursor, primitives
    guid.rs               # GUID pack/unpack (mixed-endian)
    filetime.rs           # Windows FILETIME <-> SystemTime

  types/                  # Newtypes and common data structures
    mod.rs                # SessionId, TreeId, FileId, MessageId, CreditCharge
    flags.rs              # Bitflag types (Capabilities, SecurityMode, etc.)
    status.rs             # NtStatus enum (from MS-ERREF)

  msg/                    # Wire format message structs
    mod.rs                # Command enum, Header, ErrorResponse
    header.rs             # SMB2 packet header (sync + async variants)
    negotiate.rs          # NegotiateRequest/Response, negotiate contexts
    session_setup.rs      # SessionSetupRequest/Response
    logoff.rs             # LogoffRequest/Response
    tree_connect.rs       # TreeConnectRequest/Response
    tree_disconnect.rs    # TreeDisconnectRequest/Response
    create.rs             # CreateRequest/Response, create contexts
    close.rs              # CloseRequest/Response
    flush.rs              # FlushRequest/Response
    read.rs               # ReadRequest/Response
    write.rs              # WriteRequest/Response
    lock.rs               # LockRequest/Response
    ioctl.rs              # IoctlRequest/Response
    query_directory.rs    # QueryDirectoryRequest/Response
    change_notify.rs      # ChangeNotifyRequest/Response
    query_info.rs         # QueryInfoRequest/Response
    set_info.rs           # SetInfoRequest/Response
    echo.rs               # EchoRequest/Response
    cancel.rs             # CancelRequest
    oplock_break.rs       # OplockBreakNotification/Acknowledgment
    transform.rs          # TransformHeader (encryption), CompressionTransformHeader
    dfs.rs                # DFS referral request/response wire format

  transport/              # Transport abstraction
    mod.rs                # Transport trait (split send/receive)
    tcp.rs                # Direct TCP (port 445)
    mock.rs               # Mock transport for testing

  crypto/                 # Signing, encryption, key derivation
    mod.rs
    signing.rs            # HMAC-SHA256, AES-CMAC, AES-GMAC
    encryption.rs         # AES-128/256-CCM, AES-128/256-GCM
    kdf.rs                # SP800-108 key derivation

  auth/                   # Authentication
    mod.rs                # Auth trait
    ntlm.rs              # NTLM authentication (from MS-NLMP)

  rpc/                    # Named pipe RPC (MS-RPCE / NDR)
    mod.rs                # RPC PDU types, NDR encoding/decoding
    srvsvc.rs             # NetShareEnumAll (list shares on a server)

  testing/                # Consumer test harness (feature-gated: `testing`)
    mod.rs                # TestServers API, embedded Docker infrastructure
    fixtures/consumer/    # Consumer Docker fixtures, embedded via include_str! (shipped in the crate)
    CLAUDE.md

  fuzzing.rs              # Parse entry points exposed under the `fuzzing` feature (used by `fuzz/`)

  client/                 # High-level client API
    mod.rs                # SmbClient (entry point)
    connection.rs         # Connection state, credit management, response demux
    session.rs            # Session (authenticated context)
    tree.rs               # TreeConnect (share access)
    file.rs               # Single-file convenience methods
    pipeline.rs           # Unified operation pipeline
    directory.rs          # Directory listing helpers
    shares.rs             # Share enumeration (IPC$ + srvsvc RPC)
    dfs.rs                # DFS referral IOCTL, DfsResolver with TTL cache

tests/
  pack_roundtrip.rs       # Property-based tests for pack/unpack
  msg_wire_format.rs      # Test messages against known byte sequences
  protocol_flow.rs        # Negotiate -> session -> tree -> file flows (mock)
  integration.rs          # Tests against real NAS/Pi (#[ignore])
  docker_integration.rs   # Tests against Docker Samba containers (#[ignore])
  consumer_integration.rs # Tests against consumer Docker containers (#[ignore])
  docker/                 # Docker infrastructure for smb2's own integration tests
    internal/             # Internal-suite containers (consumer fixtures live in src/testing/fixtures/)

examples/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vdavid/smb2](https://github.com/vdavid/smb2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
