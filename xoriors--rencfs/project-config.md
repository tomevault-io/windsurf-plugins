---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**rencfs** is an encrypted file system written in Rust that mounts with FUSE on Linux. It creates encrypted directories that can be safely backed up to untrusted servers or cloud storage services. The project is currently in active development and supports Linux only.

**Key differentiators**: Fast seek operations, parallel writes, memory-safe encryption key handling, modular design for library usage, and comprehensive security features including metadata encryption.

## Common Development Commands

### Building
- `cargo build` - Build for debug
- `cargo build --release` - Build for release
- `cargo build --all-targets --all-features` - Build all targets with all features

### Testing
- `cargo test --release --all --all-features` - Run all tests
- `cargo test --release` - Run tests in release mode
- `cargo bench --workspace --all-targets --all-features -j 14` - Run benchmarks
- **Python Integration Tests**: Use `tests/python/` for comprehensive file operation testing

### Code Quality
- `cargo fmt --all` - Format code
- `cargo fmt --all -- --check` - Check formatting
- `cargo clippy --all-targets --release` - Run clippy linter
- `cargo check --all` - Check compilation

### Development Workflow
Use the comprehensive check script before committing:
```bash
./scripts/check-before-push.sh
```

This script runs formatting, building, linting, testing, benchmarking, and documentation generation for both main project and java-bridge.

### Documentation
- `cargo doc --workspace --all-features --no-deps` - Generate documentation

### Running the Application
- `cargo run --release -- mount --mount-point MOUNT_POINT --data-dir DATA_DIR`
- `RENCFS_PASSWORD=password cargo run --release -- mount -m MOUNT_POINT -d DATA_DIR` (dev mode)
- `cargo run --release -- --log-level DEBUG mount -m MOUNT_POINT -d DATA_DIR` (with debug logging)
- `cargo run --release -- passwd --data-dir DATA_DIR` - Change password

### Docker Usage
- **Quick try**: `docker pull xorio42/rencfs`
- **Development**: Use provided Dockerfiles for Alpine-based builds
- **Container setup**: Requires `--device /dev/fuse --cap-add SYS_ADMIN --security-opt apparmor:unconfined`

## Code Architecture

### Technology Stack
- **Async Runtime**: Built on Tokio and fuse3 crate
- **Cryptography**: Ring crate for AEAD encryption, Argon2 for key derivation
- **Random Generation**: rand_chacha for secure randomness
- **Memory Safety**: shush-rs for secure key handling with mlock/mprotect/zeroize
- **Hashing**: Blake3 for fast cryptographic hashing
- **Password Storage**: OS keyring integration
- **Logging**: Tracing crate for structured logging

### Core Components

- **src/encryptedfs.rs** - Main encrypted filesystem implementation (`EncryptedFs`)
- **src/crypto.rs** - Cryptographic operations and cipher implementations
- **src/mount.rs** - FUSE mounting functionality (Linux-specific)
- **src/main.rs** - CLI entry point (Linux-only, shows platform messages for others)
- **src/lib.rs** - Library API with comprehensive examples

### Key Modules

- **crypto/** - Encryption/decryption with read/write streams supporting chunked operations
- **encryptedfs/** - File system operations and metadata management with inode-based structure
- **mount/** - FUSE integration and mount point management (Linux + dummy implementations)
- **keyring.rs** - OS keyring integration for password storage
- **async_util.rs** - Async utilities and helpers
- **fs_util.rs** - Filesystem utility functions
- **stream_util.rs** - Stream processing utilities
- **expire_value.rs** - Time-based value expiration for security

### Functional Design

**Data Storage Structure:**
- Encrypted data stored in dedicated directory with inode-based file structure
- All metadata, content, and filenames encrypted
- Master encryption key encrypted with password-derived key
- 256KB chunks for efficient seeking and parallel operations

**Security Features:**
- Password stored in OS keyring, cleared from memory on inactivity
- Encryption keys are mlock'd, mprotect'd, and zeroized when not in use
- Fast seek operations for instant video/media playback
- Multiple parallel writes to same file supported
- WAL (Write-Ahead Logging) for crash recovery [WIP]

**File Operations:**
- Fast seek on both reads and writes
- Chunk-based encryption (256KB chunks)
- Parallel write operations
- Instant file positioning for media files

### Supported Ciphers
- `ChaCha20Poly1305` (default) - Better for SIMD, constant-time software implementation
- `Aes256Gcm` - Hardware accelerated on most CPUs via AES-NI (1.28x faster when accelerated)

**Cipher Selection Guidelines:**
- Use AES-GCM with hardware acceleration for performance
- Use ChaCha20-Poly1305 for constant-time security without hardware acceleration
- Both are 256-bit security level with 96-bit nonces

## Testing Strategy

### Unit Tests
Tests are located in:
- `src/crypto/read/test.rs` - Crypto read operations
- `src/crypto/write/test.rs` - Crypto write operations  
- `src/encryptedfs/test.rs` - Filesystem operations
- `src/test_common.rs` - Shared testing utilities

### Integration Tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xoriors) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
