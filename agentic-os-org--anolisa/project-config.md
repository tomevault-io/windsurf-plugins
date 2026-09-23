---
trigger: always_on
description: This file provides context for AI coding assistants (Qoder, Claude, etc.) working in this repository.
---

# AGENTS.md

This file provides context for AI coding assistants (Qoder, Claude, etc.) working in this repository.

For a directory-by-directory map, see [Repository layout](README.md#repository-layout)
([中文](README_zh.md#仓库目录)). Start here for contribution rules, then follow the
component paths below to any scoped `AGENTS.md`. Source location and component
identity are separate: directory moves must preserve package names, release tags,
and existing build and test entry points.

## 1. Project Overview

**ANOLISA** is a monorepo for an Agentic OS — a server-side operating layer designed for AI agent workloads.

| Component | Path | Tech | Platform |
|-----------|------|------|----------|
| **copilot-shell** (`cosh`) | `deprecated/copilot-shell/` | TypeScript / Node.js | All |
| **cosh-ng** | `src/cosh-ng/` | Rust | Linux (full); macOS (limited functionality) |
| **agent-sec-core** | `src/agent-sec-core/` | Rust + Python | Linux only |
| **agentsight** | `src/agentsight/` | Rust (eBPF) | Linux (full); macOS (trajectory/serve only) |
| **tokenless** | `src/tokenless/` | Rust | Linux (full); macOS x64/arm64 (CLI binaries + adapters, via npm) |
| **agent-memory** (`memory`) | `src/agent-memory/` | Rust | Linux only |
| **os-skills** | `src/os-skills/` | Python / Shell | All |
| **anolisa** | `distribution/anolisa/` | Rust | Linux + macOS (arm64) |
| **SkillFS** (`skillfs`) | `src/skillfs/` | Rust / FUSE | Linux only |
| **ws-ckpt** | `src/ws-ckpt/` | Rust + TypeScript | Linux only |
| **ktuner** | `src/ktuner/` | Rust | Linux only |
| **blaze** | `src/blaze/` | Rust | Linux only |

> `agent-sec-core`, `agent-memory`, `skillfs`, `ktuner`, and `blaze` require Linux. `agentsight` provides full eBPF tracing on Linux and limited trajectory collection plus the local viewer on macOS. `cosh-ng` is Linux-first and supports limited functionality on macOS. Do **not** attempt to build the Linux-only components on macOS or Windows. (tokenless ships macOS CLI binaries and framework adapters via npm, but the binaries are cross-compiled **from Linux** — building tokenless on macOS is still unsupported.)

## 2. Development Commands

```bash
# Unified build (recommended — handles deps, build, and user install)
./scripts/build-all.sh                                        # integrated default components
./scripts/build-all.sh --no-install                           # build only, skip install
./scripts/build-all.sh --ignore-deps                          # skip dependency setup and runtime verification
./scripts/build-all.sh --component cosh --component sec-core  # selected components

# Partial convenience test runner (five components; may skip unavailable suites)
./tests/run-all-tests.sh
./tests/run-all-tests.sh --filter shell   # copilot-shell only
./tests/run-all-tests.sh --filter sec     # agent-sec-core only
./tests/run-all-tests.sh --filter sight   # agentsight only

# copilot-shell (per-component)
cd deprecated/copilot-shell
make deps      # npm install + husky hooks (use make deps-ci in CI)
make build
make lint
make test

# cosh-ng (Linux full; macOS limited functionality, per-component)
cd src/cosh-ng
cargo build --workspace
cargo fmt --all -- --check
# Select the closest targeted test from src/cosh-ng/CONTRIBUTING.md.
# Full gates require a large/cross-cutting change and an explicit request.

# agent-sec-core (Linux only; Python 3.11.6 + uv, per-component)
cd src/agent-sec-core
make build-all
uv run --project agent-sec-cli python --version  # must report 3.11.6
make test         # Python + Rust sandbox + OpenClaw plugin tests

# agentsight (Linux full eBPF; macOS trajectory/serve only, per-component)
cd src/agentsight
# Linux
make build-all
make lint
make test

# macOS
make build-mac

# os-skills
cd src/os-skills   # Skill definitions are static assets, no compilation needed

# tokenless (per-component)
cd src/tokenless
make build       # tokenless + RTK + OpenClaw plugin
make lint
make test        # Rust + hooks + integration + adapters

# agent-memory (Linux only, per-component)
cd src/agent-memory
make build       # cargo build --release --locked
make fmt-check
make lint
make test        # cargo test --locked
make smoke       # end-to-end MCP stdio smoke test

# anolisa (per-component)
cd distribution/anolisa
cargo fmt --all --check
cargo clippy --all-targets --locked -- -D warnings
cargo test --locked

# ws-ckpt (Linux only, per-component)
cd src/ws-ckpt
make build       # cargo build --release + openclaw plugin
make test        # cargo test --workspace

# SkillFS (Linux only, per-component)
cd src/skillfs
cargo fmt --all --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
scripts/test.sh   # FUSE smoke test; skips itself if fuse3 or /dev/fuse is unavailable

# ktuner (Linux only, per-component)
cd src/ktuner
cargo fmt --all --check
cargo clippy --all-targets -- -D warnings
cargo test

# blaze (Linux only, per-component)
cd src/blaze
cargo fmt --all --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```

## 3. Rust Common Conventions

> Applies to these Rust components: `anolisa`, `agentsight`, `tokenless`, `agent-memory`, `skillfs`, `ktuner`, `blaze`.

### 3.1 Comment Guidelines


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentic-os-org/ANOLISA](https://github.com/agentic-os-org/ANOLISA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
