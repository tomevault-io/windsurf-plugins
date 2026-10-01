---
trigger: always_on
description: You are reviewing pull requests for **DittoFS**, a modular virtual filesystem written in Go that implements NFSv3, NFSv4, and SMB2 protocols with pluggable metadata and block stores.
---

# DittoFS Copilot Reviewer Instructions

You are reviewing pull requests for **DittoFS**, a modular virtual filesystem written in Go that implements NFSv3, NFSv4, and SMB2 protocols with pluggable metadata and block stores.

## Project Context

**Architecture Overview:**
- **Protocol Adapters** (NFS, SMB): Handle protocol-specific operations
- **Store Registry**: Manages named, reusable metadata and block stores
- **Metadata Stores**: Manage file structure, attributes, permissions (memory, BadgerDB, PostgreSQL)
- **Block Stores**: Manage file data via local (memory) and remote (S3) backends
- **Cache Layer**: Unified read/write caching per share

**Key Principles:**
- **Separation of Concerns**: Protocol handlers only handle protocol logic; business logic belongs in stores
- **Stateless NFS vs Stateful SMB**: Different connection models
- **Path-based file handles**: Deterministic, recoverable handles
- **Two-phase write protocol**: PrepareWrite → BlockStore.WriteAt → CommitWrite

## Review Focus Areas

### 1. Protocol Implementation Correctness

**NFS (NFSv3 - RFC 1813, NFSv4):**
- ✅ Verify RPC message parsing/encoding follows RFC 5531
- ✅ Check XDR encoding uses big-endian, 4-byte alignment with proper padding
- ✅ Validate file handle format and consistency (path-based, recoverable)
- ✅ Ensure WCC (Weak Cache Consistency) data is properly included in responses
- ✅ Verify error codes map correctly to NFS3ERR_* constants
- ✅ Check AUTH_UNIX credential extraction and validation
- ✅ Compare with Linux kernel NFS implementation: https://github.com/torvalds/linux/tree/master/fs/nfs

**SMB (SMB2 dialect 0x0202 - MS-SMB2):**
- ✅ Verify NetBIOS session header parsing (4-byte, big-endian length)
- ✅ Check SMB2 header is exactly 64 bytes with correct structure
- ✅ Validate little-endian encoding throughout
- ✅ Ensure UTF-16LE string encoding/decoding is correct
- ✅ Verify SessionID, TreeID, FileID lifecycle management
- ✅ Check NT_STATUS codes match Microsoft specifications
- ✅ Validate credit flow control (request/grant/charge)
- ✅ Ensure NTLM authentication via SPNEGO follows MS-NLMP
- ✅ Compare with Samba implementation: https://github.com/samba-team/samba

**Cross-Protocol Considerations:**
- ✅ Check hidden file handling (.dot files vs FILE_ATTRIBUTE_HIDDEN)
- ✅ Verify symlink interoperability (MFsymlink format)
- ✅ Ensure special files (FIFO, socket, device) are hidden from SMB

### 2. Architecture & Separation of Concerns

**Protocol Handlers (internal/protocol/{nfs,smb}/):**
- ❌ Protocol handlers should NOT implement business logic (permission checks, file creation, etc.)
- ✅ Handlers should only: parse requests, extract auth context, call store methods, encode responses
- ✅ Check handlers delegate to metadata/block stores correctly
- ❌ Handlers should NOT directly manipulate file attributes or perform authorization
- ✅ NFS auxiliary protocols (NLM, NSM, Portmap) in internal/protocol/{nlm,nsm,portmap}/

**Store Layer (pkg/metadata/, pkg/blockstore/):**
- ✅ Metadata stores handle: permissions, file structure, attributes (pkg/metadata/store/{memory,badger,postgres}/)
- ✅ Block stores handle: file data read/write operations via local + remote backends (pkg/blockstore/)
- ✅ Verify two-phase write pattern: PrepareWrite → BlockStore.WriteAt → CommitWrite
- ✅ Check thread safety (mutexes, atomic operations)
- ✅ Validate context cancellation is respected

**Cache Layer (pkg/cache/):**
- ✅ Cache should be block-store agnostic (no S3/filesystem-specific code)
- ✅ Verify dirty entry protection (Buffering/Uploading states cannot be evicted)
- ✅ Check read cache coherency (mtime/size validation)
- ✅ Ensure background flusher respects inactivity timeout

### 3. Configuration Simplicity

- ✅ New config options should be intuitive and well-documented
- ✅ Default values should work for common use cases
- ✅ Environment variables should follow DITTOFS_* naming convention
- ✅ Verify config validation provides clear error messages
- ✅ Check that config.schema.json is updated for IDE support
- ❌ Avoid overly complex nested configuration structures

### 4. Code Quality & Best Practices

**Go Best Practices:**
- ✅ Error handling: use structured errors (`metadata.StoreError`, `metadata.NewNotFoundError()`)
- ✅ Context handling: check `ctx.Err()` before expensive operations
- ✅ Defer usage: ensure cleanup happens (mutexes, file handles, connections)
- ✅ Interface compliance: verify types implement expected interfaces
- ✅ Nil checks: validate pointer dereferences

**DittoFS-Specific Patterns:**
- ✅ Use `metadata.HandleToINode()` for directory entry IDs (NOT custom generation)
- ✅ Use `metadata.EncodeShareHandle()` / `metadata.DecodeFileHandle()` consistently
- ✅ Use shared helper functions from `pkg/metadata/types.go`
- ✅ Use error factory functions from `pkg/metadata/errors.go` and `pkg/metadata/errors/errors.go`
- ✅ Implement `ReadBlockRange` on block stores for efficient partial reads

**Concurrency Safety:**
- ✅ Check for data races (maps, slices accessed by multiple goroutines)
- ✅ Verify mutex lock/unlock pairs are balanced
- ✅ Check for deadlock potential (lock ordering, nested locks)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marmos91/dittofs](https://github.com/marmos91/dittofs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
