---
trigger: always_on
description: **Apache Iggy (Incubating)** is a persistent message streaming platform
---

# Apache Iggy Agent Guidelines

**Apache Iggy (Incubating)** is a persistent message streaming platform
in Rust. Thread-per-core shared-nothing, `io_uring` + `compio`.
Transports: QUIC, WebSocket, TCP (custom binary), HTTP (REST). SDKs:
Rust, .NET, Java, Python, Go, C++, Node.js. A connectors subsystem
ingests from / egresses to external systems via dlopened plugins.

> Skills under `.claude/skills/` are currently scoped to the
> **connectors** subsystem (`core/connectors/`). Load
> [connectors-overview](.claude/skills/connectors-overview/SKILL.md)
> first for any change there. Other subsystems follow the repo-wide
> principles in this file.

## Contents

- [Apache Iggy Agent Guidelines](#apache-iggy-agent-guidelines)
  - [Contents](#contents)
  - [STOP and ask the user before](#stop-and-ask-the-user-before)
  - [Quick reference](#quick-reference)
  - [Structure](#structure)
  - [Where to look](#where-to-look)
  - [Skills](#skills)
  - [Repo-wide principles](#repo-wide-principles)
  - [Tooling](#tooling)
  - [Testing](#testing)
  - [Pitfalls](#pitfalls)
  - [Local state directories](#local-state-directories)
  - [Commits, PRs, releases](#commits-prs-releases)
  - [Discussion and support](#discussion-and-support)

## STOP and ask the user before

- Bumping `iggy_connector_sdk` MAJOR version or changing any FFI
  signature in `core/connectors/sdk/src/{sink,source}.rs` (breaks
  every pre-built plugin .so).
- Modifying `core/binary_protocol/` wire-format types tagged
  `#[repr(C)]` (breaks every running client).
- Touching the state-save path (`core/connectors/runtime/src/state.rs`)
  in a way that changes the atomic-rename protocol (corruption risk).
- Renaming default consumer groups, plugin path resolution rules, or
  any other operator-facing identifier.
- Force-pushing to `master`, amending shipped commits, or skipping
  hooks with `--no-verify` / `--no-gpg-sign`.

## Quick reference

```bash
# Verification order is enforced by CI (.github/actions/rust/pre-merge).
cargo fmt --all                                            # step 1
cargo sort --no-format --workspace                         # step 2 (--no-format mandatory)
cargo clippy --all-features --all-targets -- -D warnings   # step 3
cargo test -p <crate>                                      # step 4

# Other CI checks (mirror in pre-commit; pass --fix where supported)
./scripts/ci/taplo.sh             # TOML
./scripts/ci/markdownlint.sh      # Markdown
./scripts/ci/shellcheck.sh        # Shell
./scripts/ci/license-headers.sh   # Apache headers
./scripts/ci/trailing-whitespace.sh
./scripts/ci/trailing-newline.sh
./scripts/ci/sync-rustc-version.sh
./scripts/ci/python-sdk-version-sync.sh
./scripts/ci/uv-lock-check.sh
./scripts/ci/binary-artifacts.sh --check
./scripts/ci/third-party-licenses.sh --validate --manifest <path>

# Run things
cargo run --bin iggy-server
cargo run --bin iggy -- <subcommand>
IGGY_CONNECTORS_CONFIG_PATH=... cargo run --bin iggy-connectors
./scripts/run-bdd-tests.sh
./scripts/run-benches.sh
```

Pre-commit hooks wire the same scripts. Install with `pre-commit
install` (Python) or `prek install` (Rust drop-in) - both read
`.pre-commit-config.yaml`.

## Structure

```text
iggy/
├── core/
│   ├── server/           Iggy server binary
│   ├── server-ng/        Next-gen server (Viewstamped Replication, WIP)
│   ├── sdk/              Rust client SDK
│   ├── cli/              iggy CLI
│   ├── connectors/       Connectors runtime + SDK + sinks/sources
│   ├── common/           Shared types (IggyDuration, IggyTimestamp, ...)
│   ├── binary_protocol/  Wire format (stable)
│   ├── configs/          Config plumbing + ConfigEnv derive
│   ├── ai/mcp/           MCP server
│   ├── bench/            Benchmark suite
│   ├── integration/      Cross-crate integration tests
│   ├── simulator/        Deterministic simulator
│   ├── consensus/        Raft (Miri-checked unsafe iobuf)
│   ├── journal/          WAL / journal
│   ├── message_bus/      In-process bus
│   ├── shard/            Shard runtime
│   ├── harness_derive/   #[iggy_harness] proc macro
│   └── metadata, partitions, server_common, configs_derive, clock, tools
├── foreign/              Java / .NET / Python / Go / C++ / Node.js SDKs
├── bdd/                  Cross-language BDD (Gherkin)
├── examples/             Per-SDK runnable examples
├── helm/                 Helm charts
├── web/                  Web UI
└── scripts/              CI + release + dev scripts
```

## Where to look

| Task                  | Location                                 |
| --------------------- | ---------------------------------------- |
| Wire protocol         | `core/binary_protocol/`                  |
| Server                | `core/server/src/`                       |
| Next-gen server (WIP) | `core/server-ng/`                        |
| Rust client SDK       | `core/sdk/src/`                          |
| Connectors            | `core/connectors/` -> connector-* skills |
| Integration tests     | `core/integration/tests/`                |
| Cross-SDK BDD         | `bdd/`                                   |
| Benchmark suite       | `core/bench/`                            |

## Skills

Connectors-scoped. Each `SKILL.md` has YAML frontmatter (name,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/iggy](https://github.com/apache/iggy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
