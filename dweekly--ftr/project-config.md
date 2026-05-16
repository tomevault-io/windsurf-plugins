---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ftr** (Fast TRaceroute) is a high-performance parallel ICMP traceroute written in Rust. It sends probes concurrently for ~10x speedup over traditional traceroute, with automatic ASN lookup, reverse DNS, public IP detection via STUN, and network segment classification. Available as both a CLI tool and a Rust library. Cross-platform: Linux, macOS, Windows, FreeBSD, OpenBSD.

- **Version**: 0.6.0
- **MSRV**: 1.82.0
- **Rust Edition**: 2021
- **License**: MIT
- **Repo**: github.com/dweekly/ftr

## Build & Test Commands

```bash
cargo build                              # Debug build
cargo build --release                    # Release build (LTO, stripped)
cargo test                               # All tests
cargo test --lib                         # Unit tests only
cargo test <test_name>                   # Single test by name
cargo test --test <file>                 # Single integration test file
cargo fmt -- --check                     # Check formatting (CI enforced)
cargo clippy -- -D warnings             # Lint — warnings are errors in CI
cargo audit                              # Security vulnerability check
cargo machete                            # Unused dependency check
cargo doc --no-deps --all-features       # Build docs (RUSTDOCFLAGS=-Dwarnings in CI)
cargo bench                              # Benchmarks (criterion)
sudo cargo run -- google.com             # Run CLI (requires root on Unix)
sudo cargo run -- --json --max-hops 20 google.com  # JSON output
```

### Git Hooks (required)

Install before committing — they enforce fmt and clippy locally:
```bash
git config core.hooksPath .githooks
```

### Compliance & Release Scripts

```bash
.githooks/check-compliance.sh            # Full local compliance check
.githooks/release-checklist.sh           # Pre-release validation
.githooks/install-tools.sh               # Install dev tools (audit, machete, etc.)
```

## Architecture

### Core Modules (`src/`)

- **`traceroute/`** — Engine: async probe sending, parallel execution, config, result types, segment classification (LAN/ISP/TRANSIT/DESTINATION), ISP detection
- **`socket/`** — Platform-abstracted socket layer with automatic fallback (Raw ICMP -> DGRAM -> privileged UDP). Each platform has its own implementation file (`linux.rs`, `macos.rs`, `bsd.rs`, `windows.rs`). Factory pattern in `factory.rs`, manual ICMP parsing in `icmp.rs`
- **`asn/`** — ASN lookup via WHOIS API with in-memory cache
- **`dns/`** — Reverse DNS via hickory-resolver with in-memory cache
- **`public_ip/`** — Public IP detection via STUN protocol with cache
- **`enrichment/`** — Parallel enrichment service (ASN + DNS lookups on hop results)
- **`services.rs`** — Service container owning ASN, DNS, STUN services
- **`caches.rs`** — Cache management
- **`config/`** — Configuration types and builder
- **`main.rs`** — CLI entry point (clap)
- **`lib.rs`** — Library public API

### Key Patterns

- **Services container**: `Services` struct owns all external service clients; `Ftr` handle wraps it for the high-level API
- **Socket fallback chain**: Factory tries socket types in order based on platform and permissions; uses `#[cfg(target_os)]` for platform-specific code
- **Shared caches**: In-memory caches for DNS, ASN, and STUN results shared across probes
- **Error codes over strings**: Permission errors use OS error codes (EPERM=1, EACCES=13), not string matching

### Platform Differences

| Platform | Socket Mode | Root Required | Notes |
|----------|-------------|---------------|-------|
| macOS    | Raw ICMP    | Yes           | Raw socket access |
| Linux    | Raw/DGRAM/UDP | No (UDP via IP_RECVERR) | Unprivileged UDP traceroute supported |
| FreeBSD/OpenBSD | Raw ICMP | Yes | Requires root for all operations |
| Windows  | Win32 ICMP API | No | Uses IcmpSendEcho, not raw sockets |

## CI Pipeline

GitHub Actions runs on push/PR to main: tests (Ubuntu stable + MSRV 1.82.0, macOS, Windows), formatting, clippy, code coverage (tarpaulin -> Codecov), security audit, unused deps (machete), outdated deps, doc build, and FreeBSD VM tests.

## Clippy Configuration

Defined in `Cargo.toml` under `[lints.clippy]`: correctness and suspicious lints are denied, `unwrap_used` warns (use `expect()` instead), `module_name_repetitions` is allowed, public items require docs (`missing_docs = "warn"`).

## PR Merging

`gh pr merge` is a remote operation — the PR merges on GitHub even if the local checkout afterward fails. Never retry `gh pr merge` after a local git error. Check `gh pr view <number> --json state` first.

## Safety Rules

- **Never run `sudo`** — ask the user to run privileged commands
- **Never run `git reset` or `git push --force`**
- **Never modify `.git/config`**
- **Never delete files without explicit user approval** (local or remote)
- **Never use `--no-verify`** when committing or pushing
- **Always run `cargo fmt` and `cargo clippy -- -D warnings` before committing**
- Platform-specific tests (`#[cfg(target_os = "...")]`) are only compiled on the target platform — test on the actual platform before pushing platform-specific changes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dweekly/ftr](https://github.com/dweekly/ftr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
