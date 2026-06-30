---
trigger: always_on
description: **Project:** OpenClaw OrbStack — one-click OpenClaw AI chatbot deployment on macOS via OrbStack VM.
---

# CLAUDE.md - Project Guide for Claude Code

**Project:** OpenClaw OrbStack — one-click OpenClaw AI chatbot deployment on macOS via OrbStack VM.
**Version:** v2026.6.10 | **License:** MIT

## Architecture

```
☁️  Cloud AI (Anthropic/OpenAI/Google)  ← AI brain
     ↑ API calls
Mac ──────────────────────────────────────
└── OrbStack
    └── Ubuntu VM (openclaw-vm)
        ├── Gateway (Node.js, systemd)    ← orchestrator, NOT in Docker
        └── Docker
            ├── sandbox-common            ← code execution
            │   └── /workspace/           ← persistent volume, agent-managed deps
            └── sandbox-browser           ← Chromium
```

Gateway runs directly on VM. Docker containers are the only isolation protecting Mac files (VM has `/mnt/mac` access).

### Three-Layer Maintenance Model

| Layer | Owner | How | Can we modify? |
|-------|-------|-----|----------------|
| Docker images | Upstream OpenClaw | Follow upstream updates, don't touch | No |
| VM (`openclaw-vm`) | Us (minimal intervention) | `local/weekly-maintenance.sh` + installer/updater scripts | Yes, but keep minimal |
| `/workspace/` in sandbox | AI agent (self-managed) | Rebuild script restores deps after container recreation | Agent-autonomous |

Each layer is responsible only for itself. Diagnose and fix issues at the layer where they occur.

The sandbox Dockerfile is upstream-controlled, so custom dependencies (bun, pnpm, etc.) are installed at runtime into `/workspace/` (persistent volume). A rebuild script can restore everything after container recreation. This is the optimal approach without forking the upstream image.

**IMPORTANT: Claude Code runs on the Mac host, NOT inside the VM.**
- NEVER run commands inside the VM, access VM files, or diagnose VM processes
- NEVER use `orb`, `ssh`, or any tool to execute inside `openclaw-vm` unless user explicitly instructs
- When VM state is needed (logs, service status, sandbox output), ask the user to provide it
- All file edits, searches, and validations happen on the Mac filesystem in this repo
- This project develops *scripts that manage* the VM — it does not run inside it

## Project Structure

- `openclaw-orbstack-setup.sh` — Main entry point (8-step installer, ~750 lines)
- `lang/en.sh`, `lang/zh-CN.sh` — i18n message strings (`$MSG_*` variables)
- `templates/openclaw.json.example` — Full JSON5 config template (reference only)
- `scripts/refresh-mac-commands.sh` — Regenerate `~/bin/openclaw-*` wrappers
- `docs/` — Architecture, commands, config guide, troubleshooting, sandbox, dev guide
- `local/` — **Developer's actual runtime config (gitignored)**, see below
- `VERSION` — openclaw-orbstack project version (not OpenClaw version)

## local/ Directory (gitignored)

This directory contains the developer's **actual runtime configuration** for their VM instance. Files here are gitignored but serve as the source of truth for the running system.

| File | Purpose | Syncs To (in VM) |
|------|---------|------------------|
| `openclaw.json` | Full Gateway config (agents, models, sandbox, channels, etc.) | `~/.openclaw/openclaw.json` |
| `.env.local` | Secrets (API keys, bot tokens, Gateway auth token) | `~/.openclaw/.env` |

### Relationship: local/ vs templates/

| Directory | Role | When to Edit |
|-----------|------|--------------|
| `templates/openclaw.json.example` | Reference template with comments | When adding new config options to document |
| `local/openclaw.json` | Actual working config | When tuning your own setup |

Config section reference is in `memory/config.md`. Detailed config docs: https://docs.openclaw.ai/gateway/configuration

## Persistent Knowledge

Architecture decisions, config structure, feature status, and operational knowledge are tracked in `memory/MEMORY.md`. Check it when you need context beyond what's in this file.

## Key Facts

| Item | Value |
|------|-------|
| VM name | `openclaw-vm` |
| Gateway port | `18789` |
| Web console | `http://openclaw-vm.orb.local:18789` |
| Config (in VM) | `~/.openclaw/openclaw.json` |
| Secrets (in VM) | `~/.openclaw/.env` |
| Codex CLI auth (in VM) | `~/.codex/auth.json` — required for ChatGPT subscription path (PR #82117 fallback) |
| Node.js | 24.x LTS |
| Service | `systemctl --user` (`openclaw-gateway.service`) |
| Gateway cmd | `node dist/index.js gateway --port 18789` |

## Build / Test / Run

```bash
# Install (interactive language selection)
bash openclaw-orbstack-setup.sh

# Skip language prompt
OPENCLAW_LANG=en bash openclaw-orbstack-setup.sh

# Validate (Claude can run these)
bash -n openclaw-orbstack-setup.sh          # syntax check
shellcheck openclaw-orbstack-setup.sh       # lint
```

No automated test suite. Validation is syntax checks + shellcheck + manual testing.

### Install Strategy

**Installer** (`openclaw-orbstack-setup.sh`) — two-tier, for fresh installs:

1. **Primary**: `npm install -g openclaw@<version>` (prebuilt npm package — fast, reliable)
2. **Fallback**: `pnpm install && pnpm build && pnpm ui:build && sudo npm install -g .` (source build — only if the npm package fails on a first install, where there's no working openclaw to fall back on)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aaajiao/openclaw-orbstack](https://github.com/aaajiao/openclaw-orbstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
