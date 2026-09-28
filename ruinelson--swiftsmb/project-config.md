---
trigger: always_on
description: SwiftSMB is a Swift Package Manager library that wraps `libsmb2` to access SMB shares from Swift. It is cross-platform and aims to be compatible with Linux and Windows in addition to Apple platforms.
---

# SwiftSMB Agent Notes

## Project Overview

SwiftSMB is a Swift Package Manager library that wraps `libsmb2` to access SMB shares from Swift. It is cross-platform and aims to be compatible with Linux and Windows in addition to Apple platforms.

The user-facing cookbook lives in `README.md` (quick examples) and `docs/` (detailed guides). Keep those in sync when public APIs change.

## Project Tree

```text
.
├── libsmb2                                   # Git submodule of libsmb2.
├── Package.swift                             # Swift Package Manager manifest.
├── Sources
│   └── SwiftSMB
│       ├── Bridge                            # Internal libsmb2 bridge; no public API here.
│       │   ├── Extensions
│       │   │   ├── Int.swift                 # Extensions to `Int`.
│       │   │   ├── String?.swift             # Extensions to `String?`.
│       │   │   └── SMB.Error.swift           # SMB.Error bridge factory and check() helper.
│       │   ├── Bridge.swift                  # High-level async POSIX-like bridge calls on per-context queues.
│       │   ├── BridgeTypes.swift             # Bridge structs/enums/options nested under `extension Bridge`.
│       │   ├── Bridge-Links.swift            # Symlink read/create bridge calls.
│       │   ├── Bridge-Locks.swift            # Byte-range lock bridge calls.
│       │   ├── Bridge-Notifications.swift    # Directory change-notification bridge calls.
│       │   ├── Bridge-ShareEnum.swift        # IPC$ share enumeration bridge calls.
│       │   └── Bridge-URL.swift              # SMB URL parsing bridge calls.
│       └── PublicAPI                         # User-facing API, all organized under SMB.
│           ├── SMB.swift                     # public final class SMB; no public initializers.
│           ├── Operations.swift              # Static top-level operations: connect/listShares/parseURL.
│           ├── Configuration.swift           # Server, credentials, and connection configuration.
│           ├── Connection.swift              # Connection handle, state, and primitive bridge operations.
│           ├── Connection-Conv.swift         # Connection convenience methods built from primitives.
│           ├── Connection-Conv-Transfer.swift # Upload/download convenience methods.
│           ├── File.swift                    # OOP file handle.
│           ├── Directory.swift               # OOP directory handle.
│           ├── Directory-Conv.swift          # Directory convenience methods built from primitives.
│           ├── Notify.swift                  # AsyncSequence-based public SMB directory notifications.
│           ├── Types.swift                   # Public value types.
│           ├── Error.swift                   # Public error type.
│           ├── Error-InvalidArgument.swift   # Typed invalid-argument operations and causes.
│           ├── Error-Status.swift            # SMB.SMBStatus and SMB.SMBStatusSeverity.
│           └── Util
│               ├── Date+.swift               # Date helpers for SMB timestamp values.
│               ├── OptionSet+.swift          # Shared debug formatting helpers.
│               ├── PathValidation.swift      # Share-name and share-relative path validation.
│               ├── Protected.swift           # Mutex/NSLock-backed state wrapper for Sendable handles.
│               └── ProtectedHandle.swift     # Protected<Handle?> wrapper shared by File/Directory/Connection.
├── Tests
│   ├── SwiftSMBUnitTests                     # Unit tests (no server needed); always run in CI.
│   │   ├── Bridge
│   │   │   ├── ConnectionConfigurationTests.swift # Context configuration unit tests.
│   │   │   └── TypeTests.swift               # Value types, errors, and enum raw-value unit tests.
│   │   └── PublicAPI
│   │       └── SMBPublicAPITests.swift       # URL parsing and public value type tests.
│   └── SwiftSMBTests                         # Integration tests; need the Docker test server.
│       ├── Bridge                            # Bridge-level Samba integration tests.
│       │   ├── ConnectionTests.swift         # Context configuration and connection lifecycle tests.
│       │   ├── DirectoryTests.swift          # Directory create, remove, and list tests.
│       │   ├── FileTests.swift               # File open, read, write, seek, and stat tests.
│       │   ├── IntegrationSupport.swift      # Shared helpers and server credentials for integration tests.
│       │   ├── OpLockTests.swift             # Oplock and lease bridge tests.
│       │   └── ShareTests.swift              # Share enumeration and info tests.
│       ├── Cookbook                          # README/docs example coverage.
│       ├── PublicAPI                         # Public API integration tests.
│       │   ├── SMBConnectionAuthTests.swift  # Authentication and connection setup tests.
│       │   ├── SMBConnectionDirectoryTests.swift # Directory convenience public API tests.
│       │   ├── SMBConnectionFileTests.swift  # File convenience public API tests.
│       │   ├── SMBConnectionTransferTests.swift # Upload/download convenience public API tests.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RuiNelson/SwiftSMB](https://github.com/RuiNelson/SwiftSMB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
