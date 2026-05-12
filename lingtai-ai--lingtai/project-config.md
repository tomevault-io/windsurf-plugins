---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is 灵台

灵台 (Língtái) is a generic agent framework — an "agent operating system" providing the minimal kernel for AI agents: thinking (LLM), perceiving (vision, search), acting (file I/O), and communicating (inter-agent email). Domain tools, coordination, and orchestration are plugged in from outside via MCP-compatible interfaces.

Named after 灵台方寸山 — where 孙悟空 learned his 72 transformations. Each agent (器灵) can spawn avatars (分身) that venture into 三千世界 and return with experiences. The self-growing network of avatars IS the agent itself — memory becomes infinite through multiplication.

### Repository Scope

All Python code (kernel runtime and batteries-included wrapper) now lives in the `lingtai-kernel` repo and is published as the `lingtai` package on PyPI. This repo contains only the Go TUI and portal frontends.

- **`tui/`** — Terminal UI (Go + Bubble Tea). Launches and monitors agents from the command line. Builds to `tui/bin/lingtai-tui`.
- **`portal/`** — Web portal (Go + embedded web frontend). Provides a browser-based interface. Builds to `portal/bin/lingtai-portal`.

Neither binary has a direct Python dependency. Both communicate with Python agents exclusively through the filesystem (`.lingtai/` directory, heartbeat files, signal files). Agents are launched by the TUI via `python -m lingtai run <dir>` as a subprocess.

### Sibling repos (not in this directory)

Other LingTai components live as sibling repos under `~/Documents/GitHub/`:

- **`lingtai-kernel/`** — Python kernel + `lingtai` PyPI package (agent runtime, LLM interface, mailbox core, preset schema validation).
- **`lingtai-skill/`** — Canonical mailbox-protocol skill. Single source of truth for `skills/lingtai/SKILL.md` (the filesystem mailbox protocol). Plugin repos vendor it via `lingtai-claude-code/scripts/sync-from-canonical.sh` — edit here, sync there.
- **`lingtai-claude-code/`** — Claude Code plugin (`claude plugin add Lingtai-AI/claude-code-plugin`). Owns the SessionStart hook and the Claude marketplace manifest. The plugin cache at `~/.claude/plugins/cache/lingtai/` gets overwritten on update — don't edit that.
- **`codex-plugin/`** — OpenAI Codex CLI plugin (`./install.sh` copies into `~/.codex/skills/lingtai/` and `~/.codex/hooks.json`).
- **MCP addon repos** — sibling MCP servers wired in via the kernel's `mcp` capability. Each is a thin adapter exposing one external system as a tool surface plus a LICC inbox listener:
  - **`lingtai-imap/`** — Gmail/IMAP mail
  - **`lingtai-telegram/`** — Telegram bot API
  - **`lingtai-feishu/`** — Feishu/Lark messaging
  - **`lingtai-wechat/`** — WeChat (gewechat) messaging
- **`lingtai-fangcun/`** — standalone skill/tool component.
- **`lingtai-agora/`**, **`lingtai-web/`** — distribution and web surfaces.
- **`lingtai-ad/`** — launch/marketing materials.

## Build

```bash
# Build the TUI
cd tui && make build
# Output: tui/bin/lingtai-tui

# Build the portal (builds embedded web frontend first)
cd portal && make build
# Output: portal/bin/lingtai-portal
```

Cross-compilation targets (darwin/linux/windows, amd64/arm64) are available via `make cross-compile` in each directory.

## Releases

See `RELEASING.md` for the full process. Key points:

1. Tag and push: `git tag v0.X.Y && git push origin v0.X.Y`
2. Create GitHub release: `gh release create v0.X.Y --title "v0.X.Y" --notes "..."` (no binary assets — Homebrew builds from source)
3. Update Homebrew tap: bump version and sha256 in `huangzesen/homebrew-lingtai/lingtai-tui.rb`, commit and push

## Projects

### TUI (`tui/`)

Go + Bubble Tea terminal interface. Key facts:

- Binary name: `lingtai-tui` (never `lingtai` — that is the Python agent CLI)
- Launches agents via `python -m lingtai run <dir>` subprocess
- Communicates with running agents via filesystem only: reads `.lingtai/` metadata, heartbeat files, and signal files inside each agent working directory
- Agent discovery uses `lingtai_kernel.handshake` conventions (`is_agent`, `is_alive` checks on working directories)

### Migrations (`tui/internal/migrate/`)

Versioned, append-only, forward-only migration system. Each migration is a file `m<NNN>_<name>.go` exporting a function `func migrate<Name>(lingtaiDir string) error`. Register it in `migrate.go` by appending to the `migrations` slice and bumping `CurrentVersion`. Migrations run once per project at TUI launch (version tracked in `.lingtai/meta.json`). They can read global state (`globalTUIDir()` helper) but receive the project's `.lingtai/` dir as input. Print warnings directly with `fmt.Println` — no i18n needed since migrations run before the TUI renders.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lingtai-AI/lingtai](https://github.com/Lingtai-AI/lingtai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
