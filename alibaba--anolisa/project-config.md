---
trigger: always_on
description: This file provides guidance to AI coding assistants when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding assistants when working with code in this repository.

## Project Overview

cosh-ng (Computable Operating System Harness) is a deterministic Agent-OS interface. It provides a `cosh-cli` binary for structured JSON output:
- **CLI mode** (`cosh-cli <subsystem> <action>`): structured JSON output for AI Agents

## Build & Test Commands

```bash
cargo build --workspace          # Build all crates
cargo test --workspace           # Run all tests (unit + integration)
cargo test --package cosh-cli --test cli_integration   # Integration tests only
cargo test --package cosh-platform   # Platform crate unit tests only
cargo test --package cosh-types      # Types crate unit tests only
```

### cosh-shell Testing Strategy

cosh-shell 的 PTY 集成测试较慢（每个 spawn 子进程）。开发时使用分层策略，避免跑全量：

```bash
# 开发时：只跑单元测试（0.1s）
cargo test --package cosh-shell --lib

# 验证逻辑：跑 logic target
cargo test --package cosh-shell --test logic

# 验证协议：跑 protocol target
cargo test --package cosh-shell --test protocol

# 验证单个集成测试（0.5-2s）
cargo test --package cosh-shell --test raw_cli <test_name> -- --exact

# 验证 shell host 改动（用并行加速）
cargo test --package cosh-shell --test shell_host -- --test-threads=4

# 阶段验收才跑全量（并行）
cargo test --package cosh-shell -- --test-threads=4
```

cosh-shell 测试布局规则：

- `src/` 只放 private 纯逻辑或轻量 component tests。
- public API 多模块逻辑测试进入 logic layer，目标 target 是 `logic`。
- adapter/control protocol 测试进入 protocol layer，目标 target 是 `protocol`。
- spawn `cosh-shell` binary、scripted raw shell、approval/question card、provider handoff 进入 `raw_cli` layer。
- PTY shell host、OSC、termios、foreground/native shell 行为进入 `shell_host` layer。
- 真实 provider、manual TTY、视觉/体验验证不混入默认 cargo test gate。

布局审计入口：

```bash
crates/cosh-shell/scripts/check-layout.sh
```

该脚本必须保持通过；新增或迁移代码不能增加新的 violation group。脚本中的 registered debt 只表示迁移债务被 inventory 追踪，不代表最终验收已完成。

Prerequisites: Linux (or macOS for limited functionality), Rust 1.74+. pkg/svc commands need root/sudo. Checkpoint commands need a running ws-ckpt daemon.

## Architecture

5-crate workspace. Dependency direction: `cosh-cli` / `cosh-core` → `cosh-platform` → `cosh-types`; `cosh-shell` is standalone (no internal crate deps).

- **cosh-types**: Pure types, zero side effects. Defines `CoshResponse<T>` envelope, `CoshError` (with error codes, recoverable flag, hint), and ws-ckpt IPC protocol types.
- **cosh-platform**: Platform abstraction layer. Distro detection from `/etc/os-release`, package manager routing (dnf/apt/zypper/brew), systemd service adapter, ws-ckpt daemon Unix socket IPC client.
- **cosh-cli**: CLI entry point (binary: `cosh-cli`). 4 command domains: `pkg`, `svc`, `checkpoint`, `audit`. All output is JSON via `CoshResponse<T>`. Uses clap derive for argument parsing.
- **cosh-core**: Unified agent core (binary: `cosh-core`). Headless JSONL backend + LLM provider integration (OpenAI-compat, SysOM/Aliyun). Includes hooks, tools, skills, extensions, and config management. Interactive TUI mode is declared but not yet implemented.
- **cosh-shell**: AI-augmented interactive shell (binary: `cosh-shell`). PTY wrapper over bash/zsh with OSC marker-based command boundary detection, streaming AI analysis (Claude/Qwen adapters), inline card rendering (ratatui), tool approval control protocol.

### cosh-shell Code Organization

每个 Issue 或任务只允许修改其 triage、已批准 design 或执行 spec 明确授权的文件；不得顺手扩大范围。
涉及 `crates/cosh-shell/` 的修改仍须遵守下列组织、owner 与布局约束。

长期 owner 约定：

- UI owner 使用 `ui/`；`agent_render/` 只允许作为短期兼容 facade。
- Hook owner 使用 `hooks/`；`hook_engine/` 合并入 `hooks/`。
- Linux memory hook 收敛到 `hooks/linux_memory/`。
- `context_window` -> `evidence/context_window.rs`。
- `exit_classify` -> `command/exit_classify.rs`。
- `governance` -> `agent/governance.rs`。
- `interactive` -> `shell_host/line_interactive.rs`。
- `hook_types` 拆分到 `types/hooks.rs` 和 `hooks/model.rs`。

新增 cosh-shell 代码时：

- 不新增 root `crates/cosh-shell/src/*.rs` implementation 文件。
- 不新增未登记的 `lib.rs pub mod` 或 root re-export。
- `src/` production code 不新增 `cosh_shell::...` self-crate public path；使用 `crate::...` 或 owner module path。
- 不向超过 1000 行的 production 文件追加新功能；超过 700 行的 production 文件需要 owner note、拆分计划或 waiver。
- `hooks` 不直接拥有 agent 启动或 runtime state mutation；通过 runtime command/event 边界交接。

## Key Design Constraints

- **ws-ckpt IPC wire format**: Uses bincode with 4-byte LE length prefix framing. Enum variant order in `WsCkptRequest`/`WsCkptResponse`/`WsCkptErrorCode` is the binary wire contract — **never reorder variants** without coordinating with the ws-ckpt daemon.
- **Unified JSON envelope**: Every CLI command returns `CoshResponse<T>` with `ok`, `data`/`error`, and `meta` fields. Exit code 0 = success, 1 = failure.
- **Cross-distro routing**: `Distro::detect()` reads `/etc/os-release` and routes to the correct package manager. Adding a new distro means adding a variant to the `Distro` enum in `cosh-platform/src/detect.rs` and updating the `pkg_manager()` method.
- **CLI helpers**: `print_success()`, `print_failure()`, `build_meta()` in `cosh-cli/src/main.rs` handle all JSON serialization and exit codes — command modules return `i32` exit codes.

## Security Heuristics


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alibaba/anolisa](https://github.com/alibaba/anolisa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
