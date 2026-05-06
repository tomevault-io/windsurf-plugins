---
trigger: always_on
description: **This file contains rules ALL agents MUST follow.** Workstream-specific instructions are in `instructions/*.md`.
---

# Nova Agent Guide

**This file contains rules ALL agents MUST follow.** Workstream-specific instructions are in `instructions/*.md`.

---

## ⚠️ CRITICAL: Memory Is the Only Hard Constraint

**Before running agents at scale, read the [Operational Guide](docs/00-operational-guide.md).**

When running hundreds of concurrent agents on a shared system (e.g., 192 vCPU / 1.5TB RAM / 110TB Disk):

| Resource | Approach |
|----------|----------|
| **CPU** | Let it burst. Scheduler handles contention fine. Use `-j$(nproc)`. |
| **Disk I/O** | Let it burst. NVMe handles parallel access. |
| **Memory** | **HARD LIMIT via RLIMIT_AS.** Exceeding = process killed (good). |

**Key Rules:**
1. **ALWAYS USE WRAPPER SCRIPTS** - `bash scripts/cargo_agent.sh` for all cargo commands
2. **NEVER RUN UNSCOPED `cargo test`** - Always include `-p/--package <crate>` or `--manifest-path <path>`
   (then optionally further scope with `--lib`, `--bin <name>`, or `--test=<name>`)
3. **GO FAST** - Use all cores for builds. Only memory matters.
4. **FAIL FAST** - Let RLIMIT_AS kill runaway processes, not the whole system.

```bash
# CORRECT:
bash scripts/cargo_agent.sh build --locked --release
bash scripts/cargo_agent.sh test --locked -p nova-core --lib

# WRONG - WILL DESTROY HOST:
cargo test
cargo build --all-targets
```

See [docs/00-operational-guide.md](docs/00-operational-guide.md) for wrapper script details.

---

## Workstreams

Each workstream has its own instruction file in `instructions/`:

| Workstream | File | Key Crates |
|------------|------|------------|
| Core Infrastructure | [`instructions/core-infrastructure.md`](instructions/core-infrastructure.md) | `nova-core`, `nova-db`, `nova-vfs`, `nova-cache`, `nova-memory` |
| Syntax & Parsing | [`instructions/syntax-parsing.md`](instructions/syntax-parsing.md) | `nova-syntax`, `nova-format` |
| Semantic Analysis | [`instructions/semantic-analysis.md`](instructions/semantic-analysis.md) | `nova-types`, `nova-resolve`, `nova-hir`, `nova-flow` |
| Code Intelligence | [`instructions/code-intelligence.md`](instructions/code-intelligence.md) | `nova-ide`, `nova-index`, `nova-fuzzy` |
| Refactoring | [`instructions/refactoring.md`](instructions/refactoring.md) | `nova-refactor` |
| Framework Support | [`instructions/framework-support.md`](instructions/framework-support.md) | `nova-framework-*`, `nova-apt` |
| Build Systems | [`instructions/build-systems.md`](instructions/build-systems.md) | `nova-build`, `nova-build-bazel`, `nova-project` |
| LSP & Editors | [`instructions/lsp-editor.md`](instructions/lsp-editor.md) | `nova-lsp`, `nova-cli`, `editors/*` |
| Debugging | [`instructions/debugging.md`](instructions/debugging.md) | `nova-dap`, `nova-jdwp` |
| AI Features | [`instructions/ai-features.md`](instructions/ai-features.md) | `nova-ai`, `nova-ai-codegen` |
| Testing & Quality | [`instructions/testing-quality.md`](instructions/testing-quality.md) | `nova-testing`, `nova-test-utils`, `fuzz/` |

**Pick your workstream and read its instruction file.** All workstream files require reading this file first.

---

## Project Overview

**Nova** is a next-generation Java Language Server that aims to surpass IntelliJ IDEA. Key innovations:

1. **Query-Based Architecture** - Salsa-inspired incremental computation
2. **Resilient by Design** - Works with broken, incomplete code
3. **Performance as a Feature** - Sub-16ms latency for most operations
4. **Composability** - Library-first design, standard protocols (LSP/DAP)

### Document Structure

| Part | Documents |
|------|-----------|
| **Operations** | [00 - Operational Guide](docs/00-operational-guide.md) (**READ FIRST**) |
| **Problem Space** | [01 - Problem Analysis](docs/01-problem-analysis.md), [02 - Current Landscape](docs/02-current-landscape.md) |
| **Architecture** | [03 - Architecture Overview](docs/03-architecture-overview.md), [04 - Incremental Computation](docs/04-incremental-computation.md), [05 - Syntax](docs/05-syntax-and-parsing.md), [06 - Semantic Analysis](docs/06-semantic-analysis.md), [16 - Java Language Levels](docs/16-java-language-levels.md) |
| **Intelligence** | [07 - Code Intelligence](docs/07-code-intelligence.md), [08 - Refactoring](docs/08-refactoring-engine.md), [09 - Framework Support](docs/09-framework-support.md) |
| **Integration** | [10 - Performance](docs/10-performance-engineering.md), [11 - Editor Integration](docs/11-editor-integration.md), [12 - Debugging](docs/12-debugging-integration.md), [17 - Observability](docs/17-observability-and-reliability.md) |
| **Advanced** | [13 - AI Augmentation](docs/13-ai-augmentation.md), [14 - Testing Strategy](docs/14-testing-strategy.md), [14 - Testing Infrastructure](docs/14-testing-infrastructure.md) |
| **Planning** | [15 - Work Breakdown](docs/15-work-breakdown.md), [Architecture + ADRs](docs/architecture.md) |

---

## Mandatory Rules (All Workstreams)

### Cargo Commands

```bash
# ALWAYS use wrapper:
bash scripts/cargo_agent.sh build --locked --release
bash scripts/cargo_agent.sh test --locked -p nova-core --lib
bash scripts/cargo_agent.sh check --locked -p nova-syntax

# NEVER run these:
cargo test                    # Unbounded
cargo build --all-targets     # Will OOM

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wilson-anysphere/indonesia](https://github.com/wilson-anysphere/indonesia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
