---
trigger: always_on
description: This document is the operating manual for coding agents working in this repository.
---

# Agent Guidelines for Avibe

This document is the operating manual for coding agents working in this repository.

## 1. Project Overview

Avibe is the local-first Agent OS: one install command turns a machine into the
runtime an agent lives in, and the user operates that runtime through Web or IM
surfaces such as Slack, Discord, Telegram, Feishu/Lark, and WeChat.

Current product shape:

- V2 config-driven service with a Web UI setup wizard and settings pages
- multi-platform message transport with shared core orchestration
- multi-backend agent routing across OpenCode, Claude Code, and Codex
- local Incus-based unified regression environment for real cross-platform verification

Default mindset:

- treat the system as **multi-platform, multi-backend** first
- prefer root-cause fixes over narrow patches
- preserve user-visible behavior unless the task explicitly changes product behavior
- make the next agent/platform inherit correct behavior automatically

## 2. Design Philosophy and Architecture

### Core Rule: Fix at the Highest Appropriate Layer

- If a bug appears on one platform, check whether the same logic exists for the others before patching a platform adapter.
- If a behavior should be shared by multiple backends, prefer the shared core or backend abstraction over a single backend implementation.
- Keep transport/platform details out of core business logic whenever possible.

Decision checklist before writing code:

1. **Scope**: is this platform-specific/backend-specific, or common?
2. **Abstraction**: can the shared base or core layer own this behavior?
3. **Call path**: is the code called from controller/handlers/common flow?
4. **Future-proofing**: would a new platform/backend inherit the correct behavior automatically?

### Codebase Map

- `main.py` - entry point wiring `config.V2Config` into `core/controller.py`
- `core/controller.py` - orchestration and dependency wiring
- `core/handlers/` - platform/backend-agnostic business workflows
- `core/message_dispatcher.py` - outbound message routing and reply enhancement flow
- `core/reply_enhancer.py` - file-link and quick-reply prompt injection helpers
- `modules/im/` - IM platform adapters (`slack.py`, `discord.py`, `telegram.py`, `feishu.py`, `wechat.py`) plus shared base classes
- `modules/agents/` - agent backend adapters (`opencode/`, `codex/`, Claude-related modules) plus shared abstractions
- `modules/im/formatters/` - platform-specific formatting built on shared formatter concepts
- `config/` - V2 config, settings, sessions, paths, and compatibility conversion
- `ui/` - React + Vite + TypeScript Web UI
- `scripts/` - operational helpers, including regression testing workflows
- `tests/` - pytest-style unit/integration/regression coverage

### Runtime Data and Important Paths

- default home: `~/.avibe/`
- legacy home: `~/.vibe_remote/` remains a compatibility path and may be a back-symlink to `~/.avibe/`
- logs: `~/.avibe/logs/vibe_remote.log`
- persisted state: `~/.avibe/state/`
- default agent working directory: `_tmp/`
- generated regression metadata: `.runtime/incus-regression/` in the primary checkout

## 3. Runtime Environments

### Local `vibe` Service

Common commands:

- install: `uv tool install avibe-os`
- run: `vibe`
- inspect: `vibe status`
- stop: `vibe stop`

Use local `vibe` for:

- local packaging checks
- local CLI behavior checks
- editable-install UI preview when explicitly needed

Hard rule:

- **Never restart the local `vibe` service for routine verification.**
- The local `vibe` process may be the coding agent runtime itself; restarting it can interrupt the session.
- **Tests and probes must be hermetic by default.** Treat `$HOME`, XDG dirs,
  keychains, CLI config/token stores, running services, browser profiles, and
  cloud accounts as production data unless the user explicitly asks otherwise.
- Any test that reaches write-capable production paths must redirect the whole
  call path to test-owned state and prove a representative write cannot touch
  real local or external user state; `uses_real_paths` tests must remain read-only.
- Unless the user explicitly asks otherwise, use the Incus regression environment for user-facing verification.

### Regression Testing (Incus)

When the user says `回归测试`, update the latest code into the existing **local**
Incus regression environment, preserve accumulated product state unless reset is
explicitly requested, then let the user verify Slack, Discord, Feishu/Lark, and
WeChat behavior.

Entry points:

- default: `./scripts/run_regression.sh`
- direct: `python3 scripts/incus_regression.py up --target master`
- macOS/Lima: `INCUS_CMD="limactl shell avibe-incus-regression -- sudo incus" ./scripts/run_regression.sh`

Hard rules:

- local Incus only for development regression; never use `--remote`, SSH, remote
  tenant projects, demos, or customer/user environments unless explicitly asked
  for remote ops
- use the runner, not raw Incus commands; it owns naming, source sync, state
  preparation, readiness checks, Show Runtime setup, metadata, and cleanup
- `master` is the long-running unified four-platform environment; keep it online,
  preserve product state, sync source, and restart the service in place

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [avibe-bot/avibe](https://github.com/avibe-bot/avibe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
