---
trigger: always_on
description: toxcore-go is a pure Go implementation of the Tox peer-to-peer encrypted messaging protocol, designed for secure communications without centralized infrastructure. The project provides a complete implementation including DHT-based peer discovery, friend management, 1-to-1 and group messaging, file transfers, audio/video calling (ToxAV), asynchronous offline messaging with forward secrecy, and identity obfuscation. It supports multi-network transports including IPv4/IPv6, Tor (.onion), I2P (.b32.
---

# Project Overview

toxcore-go is a pure Go implementation of the Tox peer-to-peer encrypted messaging protocol, designed for secure communications without centralized infrastructure. The project provides a complete implementation including DHT-based peer discovery, friend management, 1-to-1 and group messaging, file transfers, audio/video calling (ToxAV), asynchronous offline messaging with forward secrecy, and identity obfuscation. It supports multi-network transports including IPv4/IPv6, Tor (.onion), I2P (.b32.i2p), Nym (.nym), and Lokinet (.loki).

The target audience includes developers building privacy-focused communication applications, researchers working on decentralized protocols, and contributors to the Tox ecosystem. The core Go libraries have no cgo dependencies and are a pure Go solution suitable for cross-platform deployment on Linux, macOS, and Windows (amd64/arm64). Optional C API bindings are provided via the `capi` package and require cgo for cross-language interoperability.

Key differentiators include Noise Protocol Framework (IK pattern) integration for enhanced handshake security, epoch-based forward secrecy with automatic key rotation, cryptographic identity obfuscation to protect metadata from storage nodes, and automatic message padding (256B, 1024B, 4096B) to resist traffic analysis.

## Technical Stack
- **Primary Language**: Go 1.25.0 (toolchain go1.25.8), module path `github.com/opd-ai/toxcore`
- **Frameworks**:
  - `github.com/flynn/noise v1.1.0` — Noise Protocol Framework for secure handshakes (Noise-IK pattern)
  - `github.com/go-i2p/onramp v0.33.92` — I2P network transport via SAM bridge protocol
  - `github.com/opd-ai/magnum` — Opus audio codec for ToxAV (pure Go)
  - `github.com/pion/rtp v1.8.22` — RTP packet handling for audio/video streams
  - `github.com/sirupsen/logrus v1.9.4` — Structured logging with levels and fields
  - `golang.org/x/crypto v0.48.0` — Cryptographic primitives (ChaCha20-Poly1305, Curve25519, Ed25519)
  - `golang.org/x/net v0.50.0` — Network utilities
  - `golang.org/x/sys v0.41.0` — System-level operations
- **Testing**: Go's built-in `testing` package with `github.com/stretchr/testify v1.11.1` for assertions. Race detection enabled (`-race`). Coverage reported via Codecov. Build tag `nonet` excludes network-dependent tests in CI.
- **Build/Deploy**: `go build` with `gofmt` and `go vet` for code quality. CI/CD via GitHub Actions (`.github/workflows/toxcore.yml`). Cross-platform matrix builds for linux/darwin/windows on amd64 and arm64 (with `windows/arm64` explicitly excluded in the CI matrix). Semantic versioning with optional version string injection via `-ldflags` as configured in `.github/workflows/toxcore.yml` and the version variable defined in the toxcore module. No Makefile; use `go` commands directly.

## Code Assistance Guidelines

1. **Use Interface Types for Networking**: Never use concrete network types (`net.UDPConn`, `net.TCPConn`, `net.UDPAddr`, `net.TCPAddr`). Always use interface types (`net.PacketConn`, `net.Conn`, `net.Addr`, `net.Listener`). Never use type assertions or type switches to convert from interface to concrete types — use interface methods instead. This is critical for testability with mock transports (see `transport/network_transport.go`).

2. **Follow Error Wrapping Conventions**: Use `fmt.Errorf("context: %w", err)` for all error propagation. Every public function should return descriptive, wrapped errors that provide call-site context. Never silently discard errors.

3. **Implement Secure Memory Handling**: For any code touching cryptographic keys or secrets, follow the patterns in `crypto/secure_memory.go`. Wipe sensitive data from memory using `defer` statements. Use `crypto/rand` for all random number generation in security-sensitive contexts. Use constant-time comparison for cryptographic operations.

4. **Write Table-Driven Tests with Testify**: Follow the existing test patterns using `testify/assert` and `testify/require`. Name test files with suffixes: `_unit_test.go` for unit tests, `_integration_test.go` for integration tests, `_benchmark_test.go` for benchmarks. Prefer channel-based synchronization with `select`/timeout for concurrent test coordination and avoid relying on `time.Sleep` where possible (see `async/prekey_test.go` for `select`/timeout examples).

5. **Use Build Constraints for Platform-Specific Code**: Use both `//go:build` and legacy `// +build` lines. For platform-specific filesystem operations, follow the pattern in `async/storage_limits_statfs.go` (linux/darwin/BSD), `async/storage_limits_nostatfs.go` (WASM/other), and `async/storage_limits_windows.go`. The project supports `GOOS=js GOARCH=wasm` builds.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/opd-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
