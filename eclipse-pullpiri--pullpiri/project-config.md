---
trigger: always_on
description: * SPDX-FileCopyrightText: Copyright 2024 LG Electronics Inc.
---

<!--
* SPDX-FileCopyrightText: Copyright 2024 LG Electronics Inc.
* SPDX-License-Identifier: Apache-2.0
-->
# Pullpiri (Vehicle Service Orchestrator)

> **Copilot MUST apply the following prohibited/required rules as top priority.**
> 
> ### Prohibited Rules
> - Do NOT use `unwrap()`, `expect()` (no commit without code review)
> - Do NOT directly call `panic!`
> - Do NOT use `println!`, `eprintln!` in server/agent code
> - When adding external crates, license verification with cargo deny check is REQUIRED
> 
> ### Required Rules
> - Write `///` cargo doc comments for all public functions/structs
> - Errors MUST be propagated with `Result` type (do not use external crates like `anyhow`, `thiserror`)
> - No commit on build/lint/test failure or warning, MUST resolve the issue and retry
> - When code examples are requested, MUST include test code
> 
> ### Failure/Exception Handling
> - On build/lint/test failure or warning, MUST analyze the cause and retry. Do NOT ignore and commit.
> 
> ### AI Response Examples
> - When providing build instructions: Guide with `scripts/installdeps.sh` → `make build` sequence
> - When code examples are requested: MUST include doc comments and test code
> - When prohibited patterns are requested: Respond with "This pattern is prohibited by project policy"


**ALWAYS reference these instructions first. If you encounter unexpected information not covered here, you may: (1) search the project documentation, codebase, or official resources; (2) use only the bash commands explicitly listed in this document or standard diagnostic commands (e.g., `ls`, `cat`, `pwd`). If these steps do not resolve your issue, escalate by contacting a maintainer or opening an issue.**

Pullpiri is a Rust-based vehicle service orchestrator framework designed to enable efficient deployment and management of cloud-native in-vehicle services and applications. It uses a microservices architecture with server, agent, and player components that work together to orchestrate containerized workloads.

## Working Effectively

### Bootstrap and Build - NEVER CANCEL BUILDS
- **CRITICAL**: All build and dependency commands below require long timeouts. NEVER CANCEL these operations.
- Install dependencies: `scripts/installdeps.sh` -- takes 8-10 minutes. NEVER CANCEL. Set timeout to 15+ minutes.
- Build all components: `export PATH="$HOME/.cargo/bin:$PATH" && make build` -- takes 5-7 minutes. NEVER CANCEL. Set timeout to 15+ minutes.
- Format check: `export PATH="$HOME/.cargo/bin:$PATH" && scripts/fmt_check.sh` -- takes 1-2 seconds.
- Lint check: `export PATH="$HOME/.cargo/bin:$PATH" && scripts/clippy_check.sh` -- takes 2-3 minutes. NEVER CANCEL. Set timeout to 10+ minutes.

### Environment Setup Requirements
- **Operating System**: Tested on CentOS Stream 9, Ubuntu 20.04+
- **Required Dependencies**: Automatically installed by `scripts/installdeps.sh`:
  - Rust toolchain (rustup, cargo, clippy, rustfmt)
  - protobuf-compiler
  - libdbus-1-dev, libssl-dev, pkg-config
  - Docker and Docker Compose
  - cargo-deny, cargo2junit
- **External Dependencies**: Uses [Podman](https://podman.io/) container runtime
- **Ports Used**: 47001-47099 (gRPC: 47001+, REST: up to 47099), RocksDB gRPC: 47007, Settings REST: 8080


### Build Process
- **Direct Build**: Use `make build` for development builds
- **Release Build**: Use `make release` for optimized builds
- **Clean**: Use `make clean` to clean all build artifacts
- **Tools Build**: `make tools` -- builds CLI tools (pirictl, rocksdb-inspector, etc.)
- **nodeagent Binary**: `make nodeagent-bin` -- builds nodeagent with musl target for cross-compile (excluded from workspace)
- **Build Time**: Expect 5-7 minutes for full build. Dependencies download adds 2-3 minutes on first build.

### Container Operations
- **Runtime Images**: `make image` -- builds final container images for deployment (using Podman)
- **RocksDB Image**: `make rocksdb-image` -- builds RocksDB service container image
- **All Images**: `make all-images` -- builds all container images
- **Install Services**: `make install` -- deploys containers as systemd services. Requires root/sudo.
- **Uninstall Services**: `make uninstall` -- stops and removes deployed services.
- **Install/uninstall for developers**: `make dev-install` / `make dev-uninstall` -- for development environment only
- **Container Build Issues**: Container builds may fail with permission errors in some environments. Use direct Rust builds instead.

## Validation and Testing

### Pre-commit Validation - ALWAYS RUN THESE
- **ALWAYS run these commands before committing changes:**
  1. `export PATH="$HOME/.cargo/bin:$PATH" && scripts/fmt_check.sh`
  2. `export PATH="$HOME/.cargo/bin:$PATH" && scripts/clippy_check.sh`
  3. `export PATH="$HOME/.cargo/bin:$PATH" && make build`

> **Copilot Instruction**: When user requests commit creation, ALWAYS run the above validation steps (fmt_check, clippy_check, make build) first and only create commit after all pass. On validation failure, fix the issues and re-validate.

### Testing
- **Unit Tests**: `cargo test` in any crate directory (src/common, src/server, src/agent, src/player, src/tools)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eclipse-pullpiri/pullpiri](https://github.com/eclipse-pullpiri/pullpiri) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
