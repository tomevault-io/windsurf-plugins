---
trigger: always_on
description: This document provides context and instructions for AI agents working on the `Trusttunnel` codebase.
---

# Agent Guide for Trusttunnel Repository

This document provides context and instructions for AI agents working on the `Trusttunnel` codebase.

## Project Overview

TrustTunnel is an open-source VPN protocol endpoint originally developed by
AdGuard. It provides fast, secure, and reliable VPN connections whose
traffic is indistinguishable from regular HTTPS. The endpoint supports
HTTP/1.1, HTTP/2, and QUIC transports, and can tunnel TCP, UDP, and ICMP
traffic.

## Technical Context

- **Language/Version**: Rust 1.85 (edition 2021), pinned via
  `rust-toolchain.toml`
- **Primary Dependencies**: tokio, rustls, quiche (QUIC/HTTP3), h2, boring
  (BoringSSL), hyper, clap, sentry, prometheus, rcgen, instant-acme
- **Storage**: TOML configuration files on disk (`vpn.toml`, `hosts.toml`,
  `credentials.toml`, `rules.toml`); no database
- **Testing**: `cargo test` with built-in test harness; hyper and tempfile for
  integration tests; proptest for property-based tests in `deeplink`
- **Target Platform**: Linux (production), macOS (development)
- **Project Type**: Cargo workspace with 5 crates (`lib`, `endpoint`,
  `deeplink`, `macros`, `tools`)

## Project Structure

```text
vpn-libs-endpoint/
├── lib/                       # Core library crate (`trusttunnel`)
│   ├── src/                   # Protocol logic, codecs, forwarders, settings
│   │   └── authentication/    # Authentication module
│   └── tests/                 # Integration tests (tunnel, auth, ping, speedtest, reverse proxy)
│       └── common/            # Shared test helpers and fixtures
├── endpoint/                  # Binary crate (`trusttunnel_endpoint`)
│   └── src/main.rs            # CLI entrypoint for the VPN endpoint
├── tools/                     # Binary crate (`setup_wizard`)
│   └── setup_wizard/          # Interactive config generator with ACME support
├── deeplink/                  # Library crate (`trusttunnel-deeplink`)
│   └── src/                   # Deep-link URI encoding/decoding for client configs
├── macros/                    # Proc-macro crate (derive helpers)
│   └── src/                   # rt_doc, getter macros
├── bench/                     # Benchmarking scripts and Docker setup
├── scripts/                   # Automation scripts
│   ├── hooks/pre-commit       # Git pre-commit hook (lint + test)
│   ├── install.sh             # Release installation script
│   └── trusttunnel.service.template  # systemd service template
├── bamboo-specs/              # Bamboo CI pipeline definitions
├── .github/workflows/         # GitHub Actions (test, md-lint, security-audit)
├── Dockerfile                 # Multi-stage Docker build
├── docker-entrypoint.sh       # Container entrypoint script
├── Makefile                   # Development task runner
├── Cargo.toml                 # Workspace manifest
├── rust-toolchain.toml        # Pinned Rust toolchain version
├── .markdownlint.json         # Markdownlint configuration
├── vpn.toml                   # Main endpoint settings (generated, gitignored)
├── hosts.toml                 # TLS host settings (generated, gitignored)
├── credentials.toml           # User credentials (generated, gitignored)
├── rules.toml                 # Connection filtering rules (generated, gitignored)
└── certs/                     # TLS certificates for tests (generated, gitignored)
```

## Build And Test Commands

- `make init` - set up git pre-commit hooks and development environment
- `cargo build --bins` - build all binaries (debug)
- `cargo build --bins --release` - build all binaries (release)
- `make endpoint/setup ENDPOINT_HOSTNAME=<host>` - run setup wizard and
  generate `vpn.toml`/`hosts.toml` with defaults
- `make endpoint/run LOG_LEVEL=info` - build and run the endpoint with
  existing configs
- `make endpoint/gen_client_config CLIENT_NAME=<name> ENDPOINT_ADDRESS=<ip:port>` -
  generate client configuration for distribution
- `make test` - run the full test suite (`cargo test --workspace`)
- `make lint` - run all linters (`lint-rust` + `lint-md`)
- `make lint-rust` - check Rust formatting and run clippy
  (`cargo fmt --all -- --check` and `cargo clippy -- -D warnings`)
- `make lint-md` - lint Markdown files (`markdownlint .`)
- `make lint-fix` - auto-fix all linter issues
- `make lint-fix-rust` - auto-fix Rust formatting and clippy issues
- `make lint-fix-md` - auto-fix Markdown issues
- `make endpoint/clean` - clean cargo build artifacts
- `docker build -t trusttunnel-endpoint .` - build Docker image
- `docker run -it trusttunnel-endpoint` - run endpoint in Docker container

## Contribution Workflow

You MUST follow the following rules for EVERY task that you perform:

- You MUST verify your changes pass all static analysis checks before
  completing a task:
    - `make lint-rust` to check Rust formatting and run clippy
    - `make lint-md` to lint Markdown files

- You MUST update or add unit tests for any changed code.

- You MUST run the test suite to verify your changes do not break existing
  functionality:
    - `make test` (runs `cargo test --workspace`)

- When making changes to the project structure, ensure the Project Structure
  section in `AGENTS.md` is updated and remains valid.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TrustTunnel/TrustTunnel](https://github.com/TrustTunnel/TrustTunnel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
