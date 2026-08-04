---
trigger: always_on
description: This is the AdGuard DNS libraries repository, which contains a DNS proxy library that supports all existing DNS protocols including `DNS-over-TLS`, `DNS-over-HTTPS`, `DNSCrypt`, and `DNS-over-QUIC`. The library runs on Linux, macOS, Windows, and Android.
---

# AGENTS.md — Project Guide for AI Coding Agents

## Project Overview

This is the AdGuard DNS libraries repository, which contains a DNS proxy library that supports all existing DNS protocols including `DNS-over-TLS`, `DNS-over-HTTPS`, `DNSCrypt`, and `DNS-over-QUIC`. The library runs on Linux, macOS, Windows, and Android.

See [README.md](README.md) for full product details.

## Tech Stack

- **C++20** (primary), **C11** — core libraries
- **Kotlin** — Android platform adapter
- **Swift / Objective-C** — Apple platform adapter
- **CMake 3.24+** — build system
- **Conan 2.0.5+** — C++ package manager
- **Ninja** — build backend
- **Clang / LLVM 21+** — compiler and tooling

## Directory Structure

| Directory | Purpose |
| --- | --- |
| `common/` | Shared utilities: event loop, platform abstractions |
| `common/test_helpers/` | Header-only test utilities: loopback encrypted-protocol responders (`LoopbackDnsServer`, `LoopbackDnscryptServer`, `LoopbackTlsServer`, `LoopbackDohServer`, `LoopbackQuicServer`), `LoopbackHttpConnectProxy`, `MockUpstream`, DNS packet helpers (`dns_test_helpers.h`), `REQUIRE_INTEGRATION()` guard, encrypted-protocol cert kit (`test_certificates.h` + `TestCertificateVerifier` + SPKI/TBS pin helpers) |
| `net/` | Network layer: TLS, sockets, socket factory |
| `proxy/` | Core DNS proxy logic: forwarder, listener, response cache |
| `upstream/` | Upstream DNS implementations: DoH, DoT, DoQ, DNSCrypt, plain DNS |
| `dnsfilter/` | DNS filtering engine: rules, engine, filtering log |
| `dnscrypt/` | DNSCrypt client implementation |
| `dnsstamp/` | DNS stamp parsing |
| `tcpip/` | TCP/IP stack integration |
| `tools/adyg/` | `adyg`: dig-like DNS query CLI tool (plain DNS, DoT, DoH, DoQ, DNSCrypt) |
| `platform/android/` | Android adapter (Kotlin/Gradle) + standalone TUN-based DNS app |
| `platform/mac/` | Apple adapter (Swift/ObjC, CocoaPods, XCFramework) + standalone TUN-based DNS app |
| `platform/windows/` | Windows adapter (C++/CMake, C# bindings) |
| `third-party/` | Vendored dependencies: lwip, pcap_savefile, wintun |
| `scripts/` | Build helpers, Conan export, version increment, git hooks |
| `cmake/` | CMake modules: unit test helper, Conan bootstrapping/provider |
| `docs/` | Developer and platform-specific documentation |
| `.github/workflows/` | CI/CD pipeline definitions (GitHub Actions) |

### Module Dependency Flow

```text
common ← net ← upstream ← dnsproxy
common ← dnsfilter ← dnsproxy
common ← dnscrypt ← upstream
common ← dnsstamp ← upstream
common ← tcpip ← dnsproxy (optional, when DNSLIBS_ENABLE_TCPIP=ON)
common ← upstream ← adyg (standalone CLI tool in `tools/adyg`)

```

`platform/*` adapters wrap `dnsproxy` for their respective OS.

`tools/adyg` is a standalone CLI tool built directly on the `upstream` library.

## Build Commands

Run `make init` once after cloning to set up git hooks.

| Command | What It Does |
| --- | --- |
| `make init` | Configure git hooks path to `./scripts/hooks` |
| `make build_libs` | Bootstrap Conan deps → CMake configure → build `dnsproxy` |
| `make build_adyg` | Build the `adyg` dig-like DNS query CLI tool |
| `make generate_root_hints` | Regenerate `tools/adyg/root_servers.h` from the IANA root hints (needs network) |
| `make test` | Run all tests (`test-cpp`) |
| `make test-cpp` | Build libs → build test targets → run `ctest` |
| `make test-integration` | Build libs → run `ctest` with `DNSLIBS_INTEGRATION_TESTS=1` (real-network tests enabled; requires internet) |
| `make test-ci` | CI target: build libs → build test targets → run `ctest` with `DNSLIBS_INTEGRATION_TESTS=1`, `--output-junit`, and the CDash `ExperimentalTest` step. Pair with `BUILD_TYPE=debug SANITIZE=yes` on Linux to match the sanitized CI build. Note: the real-network integration tests are better suited to a scheduled build (see TODO in `Makefile`) |
| `make lint` | Run all linters (`lint-md` + `lint-cpp`) |
| `make lint-cpp` | `clang-format` check + `clangd-tidy` |
| `make lint-md` | Lint Markdown with `npx -y markdownlint-cli2@0.23.0` |
| `make lint-fix` | Auto-fix all fixable linter issues |
| `make compile_commands` | Generate `compile_commands.json` for IDE integration |
| `make clean` | Clean build artifacts |

Set `BUILD_TYPE=debug` for debug builds (default is `release` →
`RelWithDebInfo`).

## Mandatory Task Rules

You MUST follow the following rules for EVERY task that you perform:

- You MUST verify it with linter, formatter, and compiler.

  Use the following commands:
    - `make` to check if code builds
    - `make test` to build and run unit tests
    - `make lint` to run the linters
    - `make lint-fix` to fix linting issues that can be fixed automatically
    - `make clang-format` to check the formatting
    - `make clang-tidy` to run the `clang-tidy` static analysis

- You MUST prefer running Makefile targets over invoking the underlying
  tools directly. When a Makefile target exists for a task (e.g. build,
  test, lint, format, static analysis), use it rather than calling the
  tool (`clang-format`, `clang-tidy`/`clangd-tidy`, `cmake`, `ctest`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AdguardTeam/DnsLibs](https://github.com/AdguardTeam/DnsLibs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
